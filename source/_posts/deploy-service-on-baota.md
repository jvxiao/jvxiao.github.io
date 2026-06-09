---
title: 我在宝塔上部署一个 AI 图片去背景服务，踩了 4 个坑
date: 2026-06-09 21:19:28
tags: 宝塔系统, 移除图片背景
---


事情的起因很简单。

有个基于 FastAPI + rembg 的图片去背景服务，要部署到宝塔 Linux 面板上。代码只有 400 多行，逻辑也不复杂——接收图片上传、调 rembg 去除背景、返回 PNG。

我原以为十分钟搞定。结果花了三个小时。

## 第一个坑：NumPy 版本打架

第一次调接口，直接 500：

```
AttributeError: _ARRAY_API not found
A module that was compiled using NumPy 1.x cannot be run in NumPy 2.4.6
```

rembg 底层依赖 onnxruntime 和 opencv-python，这两个库在 PyPI 上分发的 Linux 预编译 wheel 是用 NumPy 1.x 的 C API 编译的。但我虚拟环境里装的是 NumPy 2.4.6，新版改了底层 C 接口，老二进制直接崩。

我下意识 `pip install "numpy<2"`，确实降下来了，但接着第二个坑就冒出来了。

后面我也试过升级 onnxruntime 和 opencv-python 到"声称支持 NumPy 2.x"的版本，升级确实拉到新版本了（onnxruntime 1.26.0、opencv 4.13.0），重启后照样报同样的 NumPy ABI 错误。原因很简单——这些新版本在 Windows 上可能用 NumPy 2.x 编译了，但 Linux 上分发的 wheel 还在用老工具链。

最后怎么解决的？直接重建虚拟环境，**让 `numpy<2` 第一个装**：

```bash
python3.11 -m venv rembg-env-v2
source rembg-env-v2/bin/activate
pip install "numpy<2"   # 必须是第一条
pip install fastapi "uvicorn[standard]" rembg pillow onnxruntime python-multipart
```

安装顺序很重要。pip 解析依赖时先锚定 numpy 1.x，后续包就会自动拉取兼容 1.x 的旧版二进制。如果倒过来装，pip 可能先选了依赖新 numpy 的包，后面再降级就冲突了。

## 第二个坑：numba 缓存罢工

numpy 降级后重启，这次换了一个错误：

```
cannot cache function '_make_tree': no locator available for file
'/www/.../pymatting/util/kdtree.py'
```

rembg 内部用了 pymatting 做抠图，pymatting 又依赖 numba 做 JIT 编译加速。numba 会在磁盘上缓存编译好的函数，numpy 版本一变，缓存里的二进制找不到对应的源文件路径了。

修起来简单：

```bash
rm -rf ~/.cache/numba
```

环境变量里顺手加上 `NUMBA_DISABLE_JIT=1`。对于这个推理服务，JIT 加速效果在一次性模型推理里几乎体现不出来，禁掉反而更稳定。

## 第三个坑：权限

然后报了第三个错误：

```
Permission denied: '/root/.u2net/tmpxlbvox0b'
```

rembg 内部用了一个叫 pooch 的库来管模型文件的缓存，默认扔在 `~/.u2net` 下面。问题来了——宝塔的 Python 项目以 `www` 用户运行，`www` 没有权限写 root 的主目录。

```bash
mkdir -p /root/.u2net
chown www:www /root/.u2net
chmod 755 /root/.u2net
```

## 第四个坑：176MB 的"惊喜下载"

权限修好之后服务终于起来了。还没来得及高兴，一看日志：

```
Downloading data from 'https://github.com/.../u2net.onnx' to file '/root/.u2net/u2net.onnx'.
0%|                                       | 16.4k/176M [00:00<39:21, 74.5kB/s]
```

服务收到第一个请求时，触发了从 GitHub 下载 176MB 模型文件。速度只有几十 KB/s。

可我明明在代码里设置了 `MODELS_OFFLINE=1`，`models/` 目录下也放了本地模型文件。

翻了一遍 rembg 源码才搞明白：rembg 在 `import` 的那一刻，内部的 pooch 就会检查 `~/.u2net/` 有没有模型文件。没有就自动下载。这个过程发生在我们自己写的任何代码之前——`MODELS_OFFLINE` 的判断、本地路径的查找，全排在它后面。

解决方案比问题本身简单得多——直接把本地模型文件扔到 pooch 的缓存目录：

```bash
cp /www/wwwroot/remove-background/models/*.onnx /root/.u2net/
chown www:www /root/.u2net/*.onnx
```

pooch 看到目录里已经有文件了，下载步骤直接跳过。

## 回头看看

四个坑，全是依赖管理和运行环境层面的问题，没有一个是服务本身的代码逻辑有 bug。

复盘下来，如果一开始就重建虚拟环境并按正确顺序安装，后三个坑里至少能省掉第一个回旋镖（numpy 反复升降级导致的 numba 缓存问题）。

最后整理一下这个服务的正确部署流程：

```bash
# 1. 建环境
python3.11 -m venv rembg-env-v2
source rembg-env-v2/bin/activate
pip install "numpy<2"
pip install fastapi "uvicorn[standard]" rembg pillow onnxruntime python-multipart

# 2. 清 numba 缓存 + 环境变量 NUMBA_DISABLE_JIT=1
rm -rf ~/.cache/numba

# 3. 修 pooch 缓存目录
mkdir -p /root/.u2net && chown www:www /root/.u2net
cp models/*.onnx /root/.u2net/
chown www:www /root/.u2net/*.onnx

# 4. 宝塔环境变量记得配
# MODELS_OFFLINE=1
# MODELS_DIR=/www/wwwroot/remove-background/models
# NUMBA_DISABLE_JIT=1
```
