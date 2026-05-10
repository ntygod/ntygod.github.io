---
title: 关于
description: 这是一处记录 AI Coding、工程实践与长期项目方法的博客站。
---

<section class="article-brief" aria-label="关于这站的摘要">
  <div>
    <span>在写什么</span>
    <p>AI Coding、长期项目工作流、测试、review 和工程实践的整理。</p>
  </div>
  <div>
    <span>怎么写</span>
    <p>尽量写成能执行、能复用、能回头验收的内容，不堆空概念。</p>
  </div>
  <div>
    <span>给谁看</span>
    <p>先给自己，也给未来回头看这套方法的人。</p>
  </div>
</section>

<section class="post-body page-panel">
  <p class="eyebrow">About</p>
  <h1>关于这站</h1>

  <p>
    这是一处偏工程向的个人博客，主要记录我在长期项目里和 AI 协作时的做法、踩坑和经验整理。
    内容会尽量围绕工作流、文档、测试、review 和长期维护来写。
  </p>

  <p>
    当前最完整的一篇是 {% if site.posts.first %}<a href="{{ site.posts.first.url | relative_url }}">在 Claude Code 里做长期项目的一套工作流</a>{% else %}暂无文章{% endif %}，
    其余内容会慢慢补到 <a href="{{ '/archive/' | relative_url }}">文章归档</a> 里。
  </p>

  <h2>写作原则</h2>
  <ul>
    <li>先把方法讲清楚，再谈感觉。</li>
    <li>优先保留可执行的东西，而不是漂亮但空的概念。</li>
    <li>不追求高频更新，追求每篇都能留下点可复用的东西。</li>
  </ul>

  <h2>站点入口</h2>
  <ul>
    <li><a href="{{ '/' | relative_url }}">首页</a></li>
    <li><a href="{{ '/archive/' | relative_url }}">归档</a></li>
    <li>{% if site.posts.first %}<a href="{{ site.posts.first.url | relative_url }}">最新文章</a>{% else %}最新文章{% endif %}</li>
  </ul>
</section>
