# ziyaochen's Blog

基于 [Jekyll](https://jekyllrb.com/) + [GitHub Pages](https://pages.github.com/) 的个人博客，
主题源自 [Hux Blog](https://github.com/Huxpro/huxpro.github.io)。

将学习中的一些心得作为记录！！

---

## 目录结构

```
.
├── _config.yml              # 站点配置
├── Gemfile                  # Ruby 依赖
├── .github/workflows/       # GitHub Actions 自动构建 + 部署
├── _includes/               # head / nav / footer 片段
├── _layouts/                # default / page / post 三个布局
├── _posts/                  # 已发布文章（YYYY-MM-DD-slug.md）
├── _drafts/                 # 草稿（不会被构建）
├── css/  js/  fonts/  img/  # 静态资源
├── index.html               # 首页（分页文章列表）
├── tags.html                # 标签页  -> /tags/
├── about.html               # 关于页  -> /about/
└── 404.html                 # 404 页
```

## 本地预览

需要 Ruby >= 3.0（macOS 系统自带的 2.6 太老，Jekyll 4 跑不起来）。

```bash
# 1. 安装一个现代 Ruby（任选其一）
brew install ruby            # 之后把 /opt/homebrew/opt/ruby/bin 加入 PATH
# 或
brew install rbenv && rbenv install 3.2.2 && rbenv local 3.2.2

# 2. 安装依赖
bundle install

# 3. 启动本地服务，带草稿和实时刷新
bundle exec jekyll serve --livereload --drafts
# 打开 http://127.0.0.1:4000
```

不想装 Ruby 的话可以用 Docker：

```bash
docker run --rm -it -v "$PWD":/srv/jekyll -p 4000:4000 \
  -e JEKYLL_ENV=development jekyll/jekyll:4 \
  jekyll serve --host 0.0.0.0 --livereload
```

## 部署

推送到 `master` 分支后，`.github/workflows/deploy.yml` 会自动构建并发布。

**首次使用需要在 GitHub 上做一次设置**：

> Settings → Pages → Build and deployment → Source 选择 **GitHub Actions**

（如果 Source 还停留在 "Deploy from a branch"，Actions 的部署步骤会直接失败。）

### 关于站点地址与 baseurl

当前仓库是 `zychen-real/ziyaochen.github.io`。GitHub 只有在仓库名等于
`<用户名>.github.io` 时才把它当作「用户主页站点」。因为仓库名是
`ziyaochen.github.io` 而用户名是 `zychen-real`，两者不一致，所以它属于**项目站点**，
访问地址是：

```
https://zychen-real.github.io/ziyaochen.github.io/
```

工作流里的 `actions/configure-pages` 会自动推导出正确的 `baseurl` 并通过
`--baseurl` 传给 Jekyll，所以两种情况都能正常工作，无需手改配置。

**建议**：把仓库重命名为 `zychen-real.github.io`，这样地址会变成简洁的
`https://zychen-real.github.io/`，`baseurl` 为空，链接也更干净。
重命名后记得把 `_config.yml` 里的 `url` 保持为 `https://zychen-real.github.io`。

## 写一篇新文章

在 `_posts/` 下新建 `YYYY-MM-DD-english-slug.md`：

```markdown
---
title:      "文章标题"
subtitle:   "副标题（可留空）"
date:       2026-09-16
tags:
    - NLP
---

正文……
```

约定：

- **文件名只用小写英文、数字和连字符**，不要有空格或中文，否则 URL 里会出现
  `%20` / 百分号编码，容易在各处链接中出错。
- `layout` / `author` / `header-img` / `catalog` / `mathjax` 已在 `_config.yml`
  的 `defaults` 里统一配置，单篇文章不需要重复写。
- 正文插图放在 `img/<文章目录>/` 下，用 `![](/img/xxx/yyy.png)` 引用。
  **注意图片扩展名大小写要和真实文件完全一致**——GitHub Pages 的服务器区分大小写，
  而 macOS 本地文件系统不区分，本地看起来正常线上却 404。
- 写作中的文章放 `_drafts/`（不带日期前缀），本地用 `--drafts` 预览，不会被发布。

## 这次整理修复了什么

| # | 问题 | 说明 |
|---|------|------|
| 1 | 站点整体 404 | 未配置 GitHub Pages 构建来源；现改为 GitHub Actions 自动部署 |
| 2 | 缺少 `Gemfile` | 无法本地预览、也无法在 CI 里锁定依赖 |
| 3 | `_config.yml` 用了废弃的 `gems:` | Jekyll 3.5 起改名为 `plugins:`，导致 `jekyll-paginate` 不加载，`paginator.posts` 为空、**首页文章列表全空白** |
| 4 | `cdn.bootcss.com` 已于 2022 年关停 | font-awesome（图标）、MathJax（公式）、highlight.js（代码高亮）全部加载失败，且是 `http://` 在 HTTPS 页面下被浏览器当混合内容拦截；已全部换成 cdnjs 的 HTTPS 地址 |
| 5 | `<script src="/js/jquery.min.js ">` 尾部多空格 | 三处 JS 引用路径末尾有多余空格 |
| 6 | `_posts/wordvecs_cs224n_2019` 文件名不合法 | 没有日期前缀也没有扩展名，Jekyll 直接忽略；内容只有一行标题，已移入 `_drafts/` |
| 7 | 6 个文章文件名含空格（其中一个是双空格） | 生成的 URL 带 `%20`；已全部重命名为英文短横线 slug |
| 8 | `img/Squicity/Result.PNG`、`OOVtest.PNG` 大小写不匹配 | 文章里引用的是 `.png`，线上 404（本地不区分大小写所以看不出来） |
| 9 | 引用了不存在的图片 | `img/favicon.ico`、`img/icon_wechat.jpg`、`img/bg/bg_404.jpg`、`img/app/*` |
| 10 | `sw.js` 是死代码 | 全项目没有任何 `serviceWorker.register` 调用，git 历史里也从未有过；连同它引用的 `offline.html` 一并删除 |
| 11 | `android_app.html` 是主题作者的遗留页 | 图片不存在、七牛云下载链接失效、导航里也没有入口 |
| 12 | 导航链接 `/about`、`/tags` 会 404 | 页面 URL 依赖 `permalink: pretty` 的隐式行为；已在两个页面里显式声明 `permalink` |
| 13 | `site.baseurl` 手工拼接 | 全站统一改用 `relative_url` / `absolute_url` 过滤器，子路径部署也不会挂 |
| 14 | 文章日期格式不统一 | `2018-7-9` 之类改为零填充；`acl2018-dialogue` 的文件名日期与 front matter 不一致，已对齐 |
| 15 | `《平凡的世界》有感` 被打上 `TASK` 标签 | 读书笔记误用了论文笔记的标签，改为 `读书` |
| 16 | MathJax 在所有页面无条件加载 | 改为只在 front matter 声明 `mathjax: true` 的页面加载 |
| 17 | 已失效的百度推送脚本 | `push.zhanzhang.baidu.com` 已下线，移除 |
