# AGENTS.md

## 项目简介

lihuiba 的个人博客「云深不知处」，基于 **Hugo**（extended 0.164.0）+ **Stack 主题**，中英文双语（默认中文），托管于 GitHub Pages：<https://lihuiba.github.io/>。评论由 Giscus 提供。

## 技术栈与部署

- 静态站点生成器：Hugo extended（CI 固定 0.164.0，见 `.github/workflows/deploy.yml`）
- 主题：`themes/hugo-theme-stack`，以 **git submodule** 引入 —— 不要直接改主题目录，用 `layouts/` 下的同名文件覆盖
- 部署：push 到 `main` 触发 GitHub Actions，`hugo --gc --minify` 构建后发布到 GitHub Pages
- 本地预览：`hugo server`（首次需 `git submodule update --init`）

## 目录结构

- `hugo.toml` —— 全部站点配置（双语、菜单、挂载、Giscus、挂件等）
- `content/post/` —— 博客文章。新文章用 page bundle 目录形式：`content/post/<slug>/index.zh.md` + `index.en.md`，图片与文章同目录
- `content/page/` —— about / archives / search 等独立页面
- `contentShared/publications/` —— 通过 `module.mounts` 同时挂载到中英两种语言的 publications 页
- `layouts/` —— 对主题的覆盖（左侧栏、RSS、Giscus、TOC、文章组件、`img` shortcode 等）
- `assets/scss/custom.scss` —— 自定义样式；`assets/icons/` 自定义图标
- `static/` —— 原样拷贝的静态文件

## 关键约定

- **双语写作**：每篇文章同时维护 `index.zh.md`（默认语言，URL 无前缀）和 `index.en.md`（URL 在 `/en/` 下）
- **front matter**：`title`、`date`、`slug`、`tags`、`description`；permalink 为 `/p/:slug/`
- **分类**：只保留 tags，不启用 categories（`hugo.toml` 中 `taxonomies` 只配了 `tag`）
- **module.mounts 陷阱**：`hugo.toml` 自定义了 mounts 后 Hugo 默认挂载全部失效，新增目录时必须显式补回对应 mount
- 提交信息风格：简短英文或中文短句，描述改动内容（参考 `git log`）

## 常用命令

```bash
git submodule update --init   # 首次克隆后拉取主题
hugo server                   # 本地预览（含草稿用 -D）
hugo --gc --minify            # 生产构建到 public/
```

## 写作偏好（博文）

规则不一致时，大体上以最新会话为优先。

- **版式**：硬换行（中文 ≤40 字/行，英文 ~72 字符）；链接不拆行；description 一句话写完整论点；tags 中文为主，技术专名保留小写英文
- **结构**：事实（带案例）→ 逐项归因 → 代价 → 残值 → 结论；开篇清单、中间分项、结论总账计数自洽；审计型文章结尾留方法论
- **证据**：关键事实带来源链接；无出处的数字不写；传闻标"据报道"；不断言不可外知之事——论证强度不超过证据强度
- **修辞**：待戳破的主张打直引号；关键结论句粗体内嵌；术语不翻译；语气客气但判词彻底
- **克制**：删节是常态，被删的展开不要补回；
- **呼应**：开篇埋清单或悬念，结尾回收；
- **内容**：事实准确，逻辑严谨，表述准确，每句话都经得起推敲；
- **身份**：资深技术专家、资深文章写手、极具洞察力；
