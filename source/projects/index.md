---
title: 我的项目
date: 2025-05-13
---

## 独立开发项目

<div class="projects-container">

## EraseBGPro

<a href="https://erasebgpro.com" target="_blank" class="project-card">

![EraseBGPro](https://erasebgpro.com/og.png)

### 🖼️ EraseBGPro

AI驱动的专业级图像背景去除工具，为电商、摄影、设计等行业提供高效的图片处理解决方案。

- **技术栈**: Vue3 + Node.js + AI图像处理
- **特性**: 批量处理、API支持、高清输出
- **访问**: [https://erasebgpro.com](https://erasebgpro.com)

</a>

## Image Toolkit

<a href="https://tools.jvxiao.cn" target="_blank" class="project-card">

![Image Toolkit](https://tools.jvxiao.cn/og.png)

### 🧰 Image Toolkit

一个功能全面的在线图片处理工具集，无需安装，随时随地处理图片。

- **技术栈**: Vue3 + Canvas/Web API
- **特性**: 图片压缩、格式转换、裁剪、滤镜等
- **访问**: [https://tools.jvxiao.cn](https://tools.jvxiao.cn)

</a>

</div>

<style>
.projects-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 24px;
  margin: 20px 0;
}

.project-card {
  display: block;
  background: var(--card-bg);
  border-radius: 12px;
  padding: 20px;
  text-decoration: none;
  color: inherit;
  transition: transform 0.3s, box-shadow 0.3s;
  border: 1px solid var(--border-color);
}

.project-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
}

.project-card img {
  width: 100%;
  height: 160px;
  object-fit: cover;
  border-radius: 8px;
  margin-bottom: 16px;
}

.project-card h3 {
  margin: 8px 0;
  font-size: 18px;
  color: var(--text-color);
}

.project-card p {
  font-size: 14px;
  color: var(--sec-text-color);
  line-height: 1.6;
  margin: 12px 0;
}

.project-card ul {
  font-size: 13px;
  color: var(--sec-text-color);
  list-style: none;
  padding: 0;
  margin: 8px 0;
}

.project-card li {
  padding: 4px 0;
}

.project-card li::before {
  content: "• ";
  color: var(--theme-color);
}
</style>