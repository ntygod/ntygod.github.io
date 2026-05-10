# Z-Code Blog

这是一个基于 GitHub Pages 的静态博客站。

## 结构

- `index.html`：首页
- `archive.html`：文章归档
- `about.md`：关于页
- `_posts/`：文章
- `_layouts/`：布局
- `assets/css/style.css`：样式
- `assets/images/hero.png`：首页封面图

## 发布

1. 把仓库推到 GitHub。
2. 在仓库 `Settings -> Pages` 里选择 `Deploy from a branch`。
3. 选择 `main` 分支和根目录 `/`。

## 写新文章

在 `_posts/` 下新增文件，命名格式：

```text
YYYY-MM-DD-title.md
```

然后在文件顶部写 front matter，正文直接用 Markdown。

## 站点风格

- 首页负责入口和最新文章。
- `archive.html` 负责全站文章索引。
- `about.md` 负责说明这站的写作范围和原则。
