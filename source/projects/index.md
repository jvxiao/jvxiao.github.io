---
title: 我的项目
date: 2025-05-13
description: jvxiao 的独立开发项目 — EraseBGPro AI背景去除、Image Toolkit 在线图片处理、薯小二 小红书博主工具箱
---

<div id="projects-app"></div>

<script>
(() => {
  const projects = [
    {
      name: 'EraseBGPro',
      url: 'https://erasebgpro.com',
      image: '/imgs/projects/erasebgpro.com.webp',
      status: 'LIVE',
      tags: ['Vue3', 'Node.js', 'AI', 'SaaS'],
      summary: 'AI驱动的专业级图像背景去除工具，为电商、摄影、设计等行业提供高效的图片处理解决方案。',
      features: ['批量处理', 'API 支持', '高清输出']
    },
    {
      name: 'Image Toolkit',
      url: 'https://tools.jvxiao.cn',
      image: '/imgs/projects/tools.jvxiao.cn.webp',
      status: 'LIVE',
      tags: ['Vue3', 'Canvas', 'Web API'],
      summary: '功能全面的在线图片处理工具集，无需安装即可完成压缩、转换、编辑等操作。',
      features: ['图片压缩', '格式转换', '裁剪 & 滤镜']
    },
    {
      name: '薯小二',
      url: 'https://xhs.jvxiao.cn',
      image: '/imgs/projects/xhs.jvxiao.cn.webp',
      status: 'LIVE',
      tags: ['Vue3', 'Node.js', 'AI', '小红书'],
      summary: '小红书博主专属创作工具箱 — 违禁词检测、AI爆款标题生成、收录检测、一键排版，让每篇笔记都能获得应有的流量。',
      features: ['敏感词检测', 'AI 标题生成', '收录检测', '文案排版']
    }
  ];

  const app = document.getElementById('projects-app');

  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('pj-visible');
        observer.unobserve(entry.target);
      }
    });
  }, { threshold: 0.1 });

  app.innerHTML = `<div class="pj-grid">${projects.map((p, i) => `
    <a href="${p.url}" target="_blank" class="pj-card" style="--pj-index: ${i}">
      <div class="pj-card-media">
        <img src="${p.image}" alt="${p.name}" loading="lazy" />
        <span class="pj-status pj-status-${p.status.toLowerCase()}">${p.status}</span>
      </div>
      <div class="pj-card-body">
        <h3 class="pj-card-title">${p.name}</h3>
        <p class="pj-card-summary">${p.summary}</p>
        <div class="pj-tags">${p.tags.map(t => `<span class="pj-tag">${t}</span>`).join('')}</div>
        <ul class="pj-features">${p.features.map(f => `<li>${f}</li>`).join('')}</ul>
      </div>
    </a>
  `).join('')}</div>`;

  app.querySelectorAll('.pj-card').forEach(card => observer.observe(card));
})();
</script>
