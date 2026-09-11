# Personal Intellectual Home — V1 设计与实施规格

> 目标：以 **as-folio** 为骨架，吸收 **Retypeset** 的长文排版气质，做一个“科研工作者 + Builder + Reader/Writer”的长期个人网站。  
> 核心原则：**首页可以酷，文章必须静；维护网站 ≈ 维护 Markdown。**

---

## 0. 一句话产品定义

这不是“博客”，而是一个长期的 **Personal Intellectual Home**：

- 对同行/博后导师：快速看到 Research / Publications / Projects / CV
- 对普通读者：看到高质量的科研经验、人生体会、文史哲长文
- 对未来的自己：形成持续增长、可搜索、可引用、可迁移的个人知识出版物

### 三重身份

1. **Scientist** — computational biology / spatial omics / bioinformatics
2. **Builder** — AI-assisted research workflows / tools / open-source projects
3. **Reader & Writer** — history / philosophy / political economy / life reflections

---

# 1. 技术路线：定稿

## 1.1 基础框架

- **Astro 6**
- **as-folio** 作为代码骨架
- **Tailwind CSS v4** 保留
- Markdown + MDX
- Astro Content Collections
- Pagefind 全文搜索
- Giscus 评论
- GitHub Actions
- GitHub Pages（V1）
- 自定义域名

### 不做

V1 不引入：

- WordPress / Ghost / 其他 CMS
- 数据库
- React 全站 hydration
- GSAP / Three.js / 大型动画库
- 复杂 headless CMS
- 用户账号
- Newsletter
- Cookie banner（除非以后启用需要 cookie 的 analytics）
- 复杂多语言 CMS

---

# 2. Sitemap

```text
/
├── writing/
│   ├── life/
│   ├── research/
│   ├── history/
│   ├── philosophy/
│   ├── society/
│   ├── tags/[tag]/
│   └── [slug]/
│
├── research/
│   ├── publications/
│   └── cv/
│
├── projects/
│   └── [slug]/
│
├── notes/
│   └── [slug]/
│
├── reading/
│
├── about/
│
├── search/
│
├── rss.xml
├── sitemap.xml
└── 404/
```

### 顶部导航最终只保留 6 项

```text
Home   Writing   Research   Projects   Notes   About
```

右侧功能：

```text
Search ⌘K   Light/Dark
```

不要把 `Publications / CV / Books / Repositories` 全塞进一级导航。

---

# 3. 首页 Wireframe

## 3.1 Desktop

```text
┌──────────────────────────────────────────────────────────────────┐
│ NAME / LOGO           Writing Research Projects Notes About  ⌘K ◐│
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  YOUR NAME / PEN NAME                                            │
│                                                                  │
│  Computational biology · Spatial omics · AI-assisted research    │
│  History · Philosophy · Political Economy                        │
│                                                                  │
│  [Research]    [Writing]    [Projects]                            │
│                                                                  │
│  GitHub · Scholar · ORCID · Email · CV                           │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  THREE IDENTITIES / Bento                                         │
│                                                                  │
│  ┌────────────────┐ ┌────────────────┐ ┌───────────────────────┐ │
│  │ SCIENTIST      │ │ BUILDER        │ │ READER & WRITER       │ │
│  │ spatial omics  │ │ AI workflows   │ │ history/philosophy    │ │
│  │ bioinformatics│ │ open source    │ │ essays/notes          │ │
│  └────────────────┘ └────────────────┘ └───────────────────────┘ │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  LATEST WRITING                                      View all →   │
│                                                                  │
│  2026.09.XX    标题                                               │
│                2-line deck / description                         │
│  2026.09.XX    标题                                               │
│  2026.09.XX    标题                                               │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  SELECTED RESEARCH                                   View all →   │
│                                                                  │
│  [2026] Paper title …                       Journal / Preprint    │
│         one-line contribution       [Paper] [Code] [Project]     │
│  [2025] ...                                                      │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  SELECTED PROJECTS                                   View all →   │
│                                                                  │
│  ┌─────────────────────┐  ┌─────────────────────┐                 │
│  │ project cover       │  │ project cover       │                 │
│  │ title               │  │ title               │                 │
│  │ 2-line description  │  │ 2-line description  │                 │
│  │ GitHub ↗            │  │ GitHub ↗            │                 │
│  └─────────────────────┘  └─────────────────────┘                 │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  NOW / READING                                                    │
│                                                                  │
│  Currently thinking about: ...                                   │
│  Reading: Book A · Book B · Journal / Long-form                  │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  Short footer                                                     │
│  © YEAR NAME · RSS · GitHub · Scholar                            │
└──────────────────────────────────────────────────────────────────┘
```

## 3.2 Mobile

顺序严格保持：

1. Hero
2. 三身份卡片（纵向）
3. Latest Writing
4. Selected Research
5. Selected Projects
6. Now / Reading
7. Footer

移动端不保留桌面 Bento 的复杂非等宽布局。

---

# 4. 首页内容策略

## Hero

建议视觉上克制，不放大头照占半屏。

左侧主文案：

```text
你的名字 / 笔名

Computational biology · Spatial omics · AI-assisted research
History · Philosophy · Political Economy
```

第二行可以有一句短 tagline，但不要写成 corporate slogan。

候选：

```text
Science, tools, histories, and notes from an unfinished education.
```

或：

```text
Researching living systems; reading histories of human systems.
```

V1 可以先使用第一种。

### CTA

三个即可：

- Research
- Writing
- Projects

不要放 6–8 个彩色按钮。

---

# 5. Writing 页面

## 5.1 列表逻辑

顶部：

```text
Writing

Long-form essays, research notes, reading reflections, and occasional
notes on life.

[All] [Life] [Research] [History] [Philosophy] [Society]
```

下面按年份排列，而不是无限卡片瀑布流。

每篇显示：

```text
2026-09-11
标题
一句 deck
History · 12 min
```

### 分类（category）只允许一个

```yaml
category:
  - life
  - research
  - history
  - philosophy
  - society
```

### Tags 可多个

例：

```yaml
tags:
  - scientific-taste
  - academic-training
  - postdoc
```

原则：

**Category = 书架；Tag = 索引。**

---

# 6. 长文 Article Page：Retypeset 化

这是整站最重要的改造。

## 6.1 页面结构

```text
                         category

              文章标题（serif, large）

           deck / subtitle（可选，1–3 行）

         2026-09-11 · 18 min · updated ...

──────────────────────────────────────────

                 正文正文正文正文

                 正文正文正文正文

       sidenote / footnote / figure / quote

                 正文正文正文正文

──────────────────────────────────────────

References / Notes

Tags

Previous / Next

Related writing

Comments
```

桌面端：

- 中央正文
- 右侧可选 sticky TOC
- 脚注可以在宽屏使用 sidenote 视觉；移动端回归标准 footnote

## 6.2 正文宽度

中文优先：

```css
--article-width: 44rem;
```

大约维持每行 35–45 个汉字，避免 Retypeset 风格被“学术主页大宽屏”破坏。

技术文章 / Distill 模式可以扩大到：

```css
--distill-width: 72rem;
```

但正文文本仍保持窄列，图表允许 breakout。

## 6.3 字号与行高

正文建议：

```text
Desktop: 18px / 1.90
Mobile: 17px / 1.85
```

英文 paragraph 可以略降低 line-height，但 V1 不必分开。

标题：

```text
H1: clamp(2.15rem, 5vw, 3.55rem)
H2: 1.65rem
H3: 1.28rem
```

段落间距：约 `1.05em–1.20em`。

---

# 7. 两种文章 Layout

## A. essay（默认）

适用：

- 人生体会
- 历史
- 哲学
- 社会
- 读书札记

Retypeset 风格：

- serif 正文
- 纸张感
- 引文
- 脚注
- 图片
- 少量 sidenote
- 安静

## B. distill

沿用 as-folio 已有 Distill 能力。

适用：

- 科研经验
- 技术教程
- AI 项目技术报告
- 数据分析
- 带公式 / 图表 / 代码 / Mermaid / notebook 的文章

允许：

- figure number
- citations
- wide figure
- code
- Mermaid
- Plotly / ECharts
- equations
- Jupyter cells

Frontmatter：

```yaml
layout: essay
# or
layout: distill
```

---

# 8. Frontmatter Schema

建议把 as-folio 的 posts schema 扩展为：

```yaml
---
title: "文章标题"
deck: "可选副标题 / 一句话摘要"
date: 2026-09-11
lastmod: 2026-09-11

category: history
tags:
  - Tang
  - political-history

lang: zh-CN

layout: essay
toc: true
comments: true

featured: false
pinned: false
draft: false
hidden: false

image: /assets/writing/example/cover.webp

description: "用于 SEO / 列表页的 100–180 字摘要"

series:
  name: "读《资治通鉴》"
  order: 1

math: false
mermaid: false
echarts: false
plotly: false
gallery: false
---
```

### 必须扩展 `src/content.config.ts`

字段：

- `deck`
- `category`
- `lang`
- `layout`
- `comments`
- `featured`
- `series`

全部以 Zod 校验。

---

# 9. Research 页面

这页不能只是论文列表。

## 页面顺序

```text
Research

1. 100–180 word research identity / current interest
2. Current / Selected Research
3. Selected Publications
4. Methods / Topics
5. Full Publications
6. CV
```

### Selected Research 卡片

每个研究项目：

```text
Spatial transcriptome–translation imaging
202X–2026

2–3 sentence scientific question / contribution

[Project story] [Paper] [Code]
```

### Publications

直接沿用：

```text
src/data/papers.bib
```

保留 as-folio：

- BibTeX
- selected
- DOI
- PDF
- code
- citation badges
- auto citation update

但视觉上降低 badge 的“仪表盘感”。

---

# 10. Projects 页面

## 分三类

```text
Research Tools
AI-assisted Research
Knowledge / Learning Systems
```

项目卡：

```text
┌─────────────────────────────────────┐
│ cover / generated preview           │
│                                     │
│ Project name                        │
│ one-sentence problem                │
│                                     │
│ Astro · Python · LLM · OCR          │
│                                     │
│ Read project →        GitHub ↗      │
└─────────────────────────────────────┘
```

### 项目详情页模板

固定结构：

1. What is it?
2. Why did I build it?
3. Design / Architecture
4. Key features
5. Screenshots / diagrams
6. What I learned
7. Limitations / next steps
8. GitHub / demo / related writing

不要把 README 原样复制过来。

---

# 11. Notes 页面

V1 定义：

> 比正式 Writing 更轻、更短、更不要求完结的公开札记。

适合：

- 读书摘记后的短想法
- 史料小考
- 某个方法的快速记录
- seminar / paper notes
- 尚未成熟成文章的思考

不建议 V1 直接把整个 Obsidian vault 公开。

### V2

以后增加：

```text
notes.your-domain.com
```

使用 Quartz，形成 Digital Garden。

主站只链接过去。

---

# 12. Reading 页面

不是 Goodreads clone。

只保留三组：

```text
Reading Now
Recently Finished
Selected / Formative Books
```

如果 as-folio Bookshelf 数据结构方便，就复用。

不要公开完整私人“待读 1000 本”队列。

---

# 13. About 页面

结构：

```text
About

portrait / optional

1. 150–250 字个人简介
2. How I work / what I care about
3. Current stage
4. Outside research
5. Links
```

内容定位：

- 不写成求职简历
- 不写成自传
- 不写政治宣言
- 让国际同行读完知道你是什么样的 researcher/person

保留：

- GitHub
- Google Scholar
- ORCID
- CV
- Email

---

# 14. 中英文策略

V1 不做“所有文章双语”。

采用：

## 英文必须有

- Home hero
- Research
- Publications
- Projects
- About
- CV

这些页面可以是英文主版或中英可切换。

## Writing

每篇文章保持原始写作语言：

```yaml
lang: zh-CN
```

或

```yaml
lang: en
```

如果某篇特别值得海外同行阅读，再单独译。

### URL

slug 永远使用稳定英文：

```text
/writing/research/scientific-taste-and-training/
```

不要：

```text
/writing/科研训练真正重要的是什么/
```

---

# 15. Visual System

## 15.1 总体风格

关键词：

```text
editorial
academic
quiet
precise
warm paper
subtle motion
scientific without cyberpunk
```

### 禁止

- neon cyberpunk
- 大面积玻璃拟态
- 动态粒子背景
- 鼠标尾迹
- 自动播放视频
- WebGL 3D 地球
- 过度渐变
- 每个 section 一种颜色

---

# 16. 色彩

## Light

```css
--bg:          #F7F6F2;
--surface:     #FFFFFF;
--text:        #202124;
--text-muted:  #697078;
--border:      #DCDDD8;

--accent:      #0F766E;
--accent-soft: #DDEDEA;
```

## Dark

```css
--bg:          #111315;
--surface:     #181B1E;
--text:        #ECEAE4;
--text-muted:  #A7ADB3;
--border:      #30353A;

--accent:      #5EEAD4;
--accent-soft: #163B38;
```

### Accent 使用规则

只用于：

- link
- active navigation
- tiny category indicator
- selected state
- subtle hover

不用于大面积背景。

---

# 17. Typography

## UI / Navigation

优先系统 sans：

```css
font-family:
  Inter,
  ui-sans-serif,
  system-ui,
  -apple-system,
  BlinkMacSystemFont,
  "Segoe UI",
  "PingFang SC",
  "Microsoft YaHei",
  sans-serif;
```

## Long-form Body

```css
font-family:
  "Source Serif 4",
  "Source Han Serif SC",
  "Noto Serif CJK SC",
  "Songti SC",
  "STSong",
  serif;
```

### V1 性能策略

**不直接在线加载完整 CJK webfont。**

先采用：

- English serif 可小体积 self-host
- 中文用系统宋体 fallback

V1.1 再根据实际设备截图决定是否：

- self-host 思源宋体 WOFF2 subset
- 按实际网站字符自动做 font subsetting

这样避免为了“字体漂亮”加载几 MB～几十 MB。

---

# 18. 组件设计

新增/重构这些组件：

```text
src/components/
├── home/
│   ├── Hero.astro
│   ├── IdentityGrid.astro
│   ├── LatestWriting.astro
│   ├── SelectedResearch.astro
│   ├── SelectedProjects.astro
│   └── NowReading.astro
│
├── writing/
│   ├── ArticleHeader.astro
│   ├── ArticleMeta.astro
│   ├── ArticleTOC.astro
│   ├── ArticleFooter.astro
│   ├── RelatedWriting.astro
│   ├── SeriesNav.astro
│   ├── Sidenote.astro
│   └── PullQuote.astro
│
├── research/
│   ├── ResearchCard.astro
│   └── PublicationList.astro
│
├── projects/
│   ├── ProjectCard.astro
│   └── ProjectMeta.astro
│
├── common/
│   ├── SectionHeader.astro
│   ├── Tag.astro
│   ├── ExternalLink.astro
│   └── EmptyState.astro
│
└── layout/
    ├── SiteHeader.astro
    ├── SiteFooter.astro
    └── ThemeToggle.astro
```

---

# 19. Layouts

保留 as-folio 的架构，但重新定义用途：

```text
src/layouts/
├── Base.astro
├── Page.astro
├── Post.astro          → essay layout
├── Distill.astro       → technical/research
└── Project.astro       → project story
```

如果 `Post.astro` 改动过大：

```text
Essay.astro
```

单独建立，避免影响上游 Distill。

---

# 20. Styles

目标结构：

```text
src/styles/
├── global.css
├── _colors.css
├── _typography.css
├── _article.css
├── _motion.css
├── _cards.css
└── _print.css
```

## Print CSS

文章必须提供良好打印 / Save as PDF 效果：

- hide nav
- hide comments
- hide related cards
- black text / white background
- links 显示 URL（可选）
- footnotes 保留

---

# 21. Motion

只用：

- Astro View Transitions
- CSS transition

规则：

```text
page fade:        150–220ms
card hover:       120–180ms
hover translate:  max 2px
```

支持：

```css
@media (prefers-reduced-motion: reduce)
```

### 不允许

- 页面滚动劫持
- parallax
- 卡片大幅弹跳
- stagger 逐个飞入
- 背景持续动画

---

# 22. Article Components

MDX 允许：

```mdx
<PullQuote>
...
</PullQuote>

<Sidenote>
...
</Sidenote>

<Figure
  src="..."
  caption="..."
  source="..."
/>

<Callout type="note">
...
</Callout>
```

默认 Markdown：

- footnotes
- blockquote
- tables
- code
- KaTeX

---

# 23. Repo 目录建议

以 as-folio 当前目录为基础：

```text
personal-site/
├── AGENTS.md
├── CLAUDE.md
├── CUSTOMIZE.md
├── README.md
│
├── src/
│   ├── config/
│   │   ├── site.ts
│   │   └── navigation.ts
│   │
│   ├── content.config.ts
│   │
│   ├── content/
│   │   ├── posts/
│   │   │   ├── life/
│   │   │   ├── research/
│   │   │   ├── history/
│   │   │   ├── philosophy/
│   │   │   └── society/
│   │   │
│   │   ├── projects/
│   │   ├── notes/
│   │   ├── books/
│   │   └── announcements/
│   │
│   ├── data/
│   │   ├── papers.bib
│   │   ├── coauthors.yml
│   │   ├── citations.yml
│   │   ├── cv.yml
│   │   ├── resume.json
│   │   └── repositories.yml
│   │
│   ├── components/
│   ├── layouts/
│   ├── pages/
│   └── styles/
│
├── public/
│   ├── assets/
│   │   ├── avatar/
│   │   ├── writing/
│   │   ├── projects/
│   │   ├── research/
│   │   └── pdf/
│   └── favicon.svg
│
├── scripts/
│   ├── update-citations.ts
│   └── validate-content.ts
│
└── .github/
    └── workflows/
        ├── deploy.yml
        └── check.yml
```

### V1 删除或隐藏的 as-folio 功能

如无内容：

- teaching
- people
- repositories 独立页面
- newsletter
- announcements 可改名为 `now` 或完全不用

**不要为了继承主题而保留空页面。**

---

# 24. 内容文件命名

```text
YYYY-MM-DD-short-english-slug.md
```

例如：

```text
2026-09-11-scientific-taste-and-training.md
```

项目：

```text
ai-translation-pipeline.md
```

Notes：

```text
2026-09-11-li-deyu-notes.md
```

---

# 25. 图片策略

每篇长文：

```text
public/assets/writing/<slug>/
```

项目：

```text
public/assets/projects/<slug>/
```

原则：

- 不把所有图片扔进一个 `img/`
- 优先 WebP / AVIF
- hero / screenshot 用 Astro image pipeline
- 明确 width / height，避免 CLS
- 技术截图不强行裁成 16:9
- 历史图像写 caption / source

---

# 26. Search

直接保留 as-folio：

```text
Pagefind + ninja-keys
```

快捷键：

```text
⌘K / Ctrl+K
```

索引：

- Writing
- Notes
- Projects
- Publications

Draft / hidden 禁止进入索引。

---

# 27. Comments

Giscus。

默认：

```yaml
comments: true
```

但可以单篇关闭。

### 开启

- Writing
- Projects

### 默认不开

- About
- Publications
- Notes（V1 建议关闭，减少维护）

---

# 28. SEO / Metadata

每篇文章必须生成：

- `<title>`
- meta description
- canonical
- OpenGraph
- Twitter/X card
- JSON-LD Article
- language
- published / modified

Research / Project：

- Person / ScholarlyArticle / SoftwareSourceCode 等 schema 可在 V1.1 做

---

# 29. Accessibility

验收要求：

- semantic HTML
- keyboard navigation
- visible focus
- contrast AA
- alt text
- reduced motion
- headings 不跳级
- touch target ≥ 44px
- 不靠颜色单独表达状态

---

# 30. Performance Budget

首页：

- JS 尽量 `< 100 KB gzip`（第三方搜索打开后不计入首屏）
- 无 autoplay video
- 无背景视频
- 图片 lazy load
- Lighthouse Performance ≥ 95（桌面）
- CLS 接近 0

文章页：

- 默认无大型 JS
- Mermaid / Plotly / ECharts 只有 frontmatter 开启时加载
- comments 到页面底部再加载

---

# 31. Git / Deployment

V1：

```text
GitHub repository
↓
GitHub Actions
↓
yarn build
↓
Pagefind index
↓
GitHub Pages
↓
custom domain
```

PR / push 前：

```bash
yarn lint
yarn test
yarn build
```

必须遵守 as-folio 原项目的 `AGENTS.md` / `CLAUDE.md`。

---

# 32. 写作工作流

推荐内容源：

```text
Obsidian/private      → 永不发布
Obsidian/publish      → 人工确认后复制/同步进 repo
```

V1 **不要自动同步整个 Vault**。

发布：

```text
write
→ set draft:false
→ git diff
→ build
→ commit
→ push
→ GitHub Action deploy
```

以后再做：

```text
publish-site script
```

自动完成：

- validate frontmatter
- optimize images
- detect broken links
- update lastmod
- build
- git commit

---

# 33. 首页初始内容配额

不要等内容很多才上线。

上线需要：

### Writing

至少 3 篇：

1. 科研经验 / scientific taste
2. 文史哲
3. 人生 / 博士阶段反思

### Research

- 个人研究简介
- 2–4 个 selected publications
- CV

### Projects

至少 3 个：

- 一个 AI-assisted research pipeline
- 一个历史/人文研究 pipeline
- 一个技术/学习系统

### About

1 页。

够了。

---

# 34. V1 → V2 路线图

## V1：可发布

- as-folio fork
- clean navigation
- new homepage
- Retypeset-like typography
- Writing taxonomy
- Research
- Projects
- About
- Pagefind
- dark mode
- Giscus
- GitHub Pages

## V1.1：精修

- OG image
- CJK font subsetting
- print CSS
- better footnotes / sidenotes
- series navigation
- reading page
- analytics（如果真的需要）

## V2

- `notes.domain.com` Quartz
- selective Obsidian publishing
- bilingual essential pages
- automatic content QA
- better scholarly citation component
- project changelog / status

---

# 35. 开发阶段划分

## P00 — Fork 与基线

目标：

- fork as-folio
- 安装
- build pass
- 删除 demo persona
- 确认上游规则

验收：

```text
yarn lint
yarn test
yarn build
```

全部通过。

---

## P01 — Information Architecture

完成：

- 顶部导航
- route
- content schema
- 删除/hide 无用页面
- categories / tags / lang / layout

不改视觉。

---

## P02 — Design Tokens

完成：

- color variables
- type scale
- serif/sans system
- spacing
- border/radius
- light/dark
- reduced motion

不改页面业务逻辑。

---

## P03 — Homepage

完成：

- hero
- identity grid
- latest writing
- selected research
- selected projects
- now/reading
- responsive

---

## P04 — Writing / Retypeset

这是重点。

完成：

- Writing index
- Essay layout
- article typography
- footnotes
- blockquotes
- TOC
- tags
- related
- previous / next
- comments
- print CSS

---

## P05 — Research / Projects / About

完成：

- Research narrative
- publications
- project grid
- project detail layout
- About

---

## P06 — Search / SEO / Performance

完成：

- Pagefind
- metadata
- OG
- RSS
- sitemap
- accessibility
- image optimization
- performance

---

## P07 — Content & Launch

完成：

- 替换 demo
- 第一批文章
- 第一批项目
- publications
- CV
- domain
- GitHub Pages deploy

---

# 36. Master Prompt — 交给 OMP / Codex

下面这段可以直接作为总任务说明。

```text
You are upgrading a personal academic/intellectual website based on the current
upstream as-folio Astro theme.

The website must NOT become a generic academic CV site and must NOT become a
flashy developer portfolio.

Product definition:
A long-term “personal intellectual home” for a computational biology researcher
who also publishes long-form writing on research practice, life, history,
philosophy, and political economy, and showcases AI-assisted research /
knowledge-work projects.

Core design rule:
“Homepage can be visually expressive; article pages must be quiet.”

Architecture:
- Astro 6
- keep as-folio as the base
- keep Tailwind v4
- Markdown/MDX + Astro Content Collections
- keep BibTeX publications
- keep Pagefind + ninja-keys
- keep dark mode
- use Giscus
- GitHub Pages
- no database
- no CMS
- no heavy animation library
- no React-wide hydration

Visual direction:
- as-folio structure and academic functionality
- Retypeset-inspired long-form typography
- editorial / academic / warm-paper
- subtle motion only
- science without cyberpunk aesthetics

Primary routes:
Home / Writing / Research / Projects / Notes / About

The homepage must communicate three identities:
Scientist / Builder / Reader & Writer.

Writing categories:
life / research / history / philosophy / society.

Implement two writing layouts:
1. essay: typography-first, serif, narrow column, footnotes, long-form reading.
2. distill: preserve as-folio technical/scientific layout for figures, code,
   equations, citations, Mermaid, charts and notebooks.

Before making any changes:
1. Read AGENTS.md.
2. Read CLAUDE.md.
3. Read CUSTOMIZE.md.
4. Read src/config/site.ts.
5. Read src/content.config.ts.
6. Run yarn build and record baseline.
7. Do not use npm or npx.
8. Do not edit yarn.lock manually.
9. Preserve existing as-folio conventions where they are sound.

Engineering requirements:
- Every new content field must be added to the Zod schema.
- Every new user-facing feature must have config where appropriate.
- Do not hardcode personal strings in reusable components.
- Keep JS minimal.
- Respect prefers-reduced-motion.
- Semantic HTML and accessible keyboard navigation.
- Production build must pass after each phase.

Work phase-by-phase. After each phase:
- summarize changed files,
- explain design/architecture decisions,
- list unresolved issues,
- run lint/test/build as applicable,
- stop and wait for review before starting the next phase.

Follow the accompanying PERSONAL_SITE_SPEC.md as the source of truth.
If an implementation choice conflicts with the current as-folio architecture,
prefer a minimal compatible extension over a rewrite.
```

---

# 37. P00 Prompt

```text
Execute P00 only.

Tasks:
1. Inspect the entire repository.
2. Read AGENTS.md, CLAUDE.md, CUSTOMIZE.md, src/config/site.ts and
   src/content.config.ts.
3. Run the current lint/test/build baseline.
4. Produce a concise repository architecture map.
5. Identify which as-folio features can be reused unchanged:
   publications, BibTeX, Pagefind, dark mode, Distill, projects, books,
   comments, RSS, sitemap, citation updater.
6. Identify demo/persona content that must be removed or replaced.
7. Do NOT redesign pages yet.
8. Create docs/IMPLEMENTATION_PLAN.md mapping P01–P07 to concrete current
   repository files.

Acceptance:
- baseline build status documented
- no unnecessary dependency changes
- no UI redesign yet
- implementation plan references actual repository paths

Stop after P00 for review.
```

---

# 38. P01 Prompt

```text
Execute P01 only: information architecture and content model.

Implement:
- top-level nav: Home / Writing / Research / Projects / Notes / About
- Writing category model: life/research/history/philosophy/society
- add frontmatter/schema fields:
  deck, category, lang, layout, comments, featured, series
- support essay/distill layout selection
- Notes content collection
- stable English slugs
- remove or hide unused Teaching/People/etc. from public navigation

Do not do visual redesign yet.

Add minimal fixture content sufficient to validate every route/schema.

Run:
yarn lint
yarn test
yarn build

Document changed files and stop for review.
```

---

# 39. P02 Prompt

```text
Execute P02 only: design token system.

Implement the visual tokens defined in PERSONAL_SITE_SPEC.md:
- warm paper light theme
- restrained dark theme
- teal accent
- UI sans stack
- long-form serif stack
- type scale
- spacing
- border/radius
- article width tokens
- motion tokens
- prefers-reduced-motion

Refactor existing color/typography CSS rather than sprinkling arbitrary
Tailwind values across components.

Do not redesign all pages yet.
Produce a small internal visual test page or use existing components to verify
tokens in light/dark and desktop/mobile.

Run lint/test/build and stop.
```

---

# 40. P03 Prompt

```text
Execute P03 only: homepage redesign.

Build:
- Hero
- IdentityGrid: Scientist / Builder / Reader & Writer
- LatestWriting
- SelectedResearch
- SelectedProjects
- NowReading
- concise footer

Requirements:
- data comes from config/content, not hardcoded demo persona strings
- subtle CSS hover only
- no hero background video/particles/WebGL
- excellent mobile stacking
- homepage should feel editorial and intellectual, not corporate or cyberpunk

Use the desktop/mobile wireframes in PERSONAL_SITE_SPEC.md.

Run lint/test/build.
Include screenshots or local preview notes for:
- desktop light
- desktop dark
- mobile light

Stop for review.
```

---

# 41. P04 Prompt

```text
Execute P04 only: Writing and Retypeset-inspired long-form reading.

This is the highest-priority visual phase.

Implement:
- Writing index with All/Life/Research/History/Philosophy/Society
- year-grouped list
- Essay layout
- article header
- narrow reading column
- serif long-form typography
- footnotes
- blockquotes
- figure/caption
- TOC
- series navigation
- related writing
- previous/next
- tags
- optional Giscus
- print CSS
- mobile footnotes

Preserve Distill layout as the technical/scientific alternative.

Do NOT copy Retypeset source code wholesale. Recreate the reading principles
within as-folio’s component/style architecture.

Test:
- Chinese long-form article
- English long-form article
- mixed Chinese/English article
- footnotes
- code block
- table
- image
- very long H2/H3 headings

Run lint/test/build and stop.
```

---

# 42. P05 Prompt

```text
Execute P05 only.

Redesign:
1. Research
   - research identity text
   - selected research stories
   - selected publications
   - full publication list
   - CV link
2. Projects
   - category groups
   - clean card layout
   - project story detail layout
3. About
   - narrative-first personal page
   - academic/social links

Reuse as-folio BibTeX and project plumbing.
Reduce badge/dashboard visual noise.

Run lint/test/build and stop.
```

---

# 43. P06 Prompt

```text
Execute P06 only: search, metadata, quality and performance.

Verify/implement:
- Pagefind indexes Writing/Notes/Projects/Publications
- drafts and hidden entries excluded
- canonical URLs
- OpenGraph
- RSS
- sitemap
- Article metadata / JSON-LD where appropriate
- image dimensions and optimization
- accessible focus states
- keyboard navigation
- reduced motion
- 404
- responsive edge cases
- no unnecessary client JS

Audit:
- mobile
- light/dark
- Chinese content
- English content
- performance
- accessibility

Run lint/test/build and report remaining launch blockers.
Stop.
```

---

# 44. P07 Prompt

```text
Execute P07 only: content replacement and launch preparation.

Remove remaining demo content and prepare actual content slots.

Do NOT invent biographical or publication facts.
Where real content is unavailable, create clearly marked TODO placeholders.

Create a launch checklist for:
- site identity
- domain
- avatar
- favicon
- About
- Research
- papers.bib
- CV
- at least 3 Writing posts
- at least 3 Projects
- GitHub / Scholar / ORCID
- Giscus repository configuration
- ASTRO_SITE / ASTRO_BASE
- GitHub Pages

Run the final lint/test/build.
Do not deploy until explicitly approved.
```

---

# 45. 最终验收 Definition of Done

## Identity

- 10 秒内能理解 Scientist / Builder / Reader & Writer
- 不像标准实验室主页
- 不像程序员求职模板
- 不像 Substack clone

## Reading

- 中文 5000–10000 字文章阅读舒适
- 手机上不挤
- footnote / quote / figure 美观
- 暗色模式长时间阅读不刺眼

## Research

- 30 秒能找到你的研究方向、代表作、CV、Scholar

## Projects

- 读者不用打开 GitHub README 就能理解项目价值

## Maintenance

发布一篇文章不需要改 `.astro` 文件。

理想发布操作：

```text
新增一个 Markdown
→ draft:false
→ push
```

## Engineering

```text
yarn lint   PASS
yarn test   PASS
yarn build  PASS
```

---

# 46. 最重要的五条“不许偏航”

1. **不要从 as-folio 重写成另一个框架。**
2. **不要为了“酷炫”牺牲长文阅读。**
3. **不要为了展示功能保留无内容的页面。**
4. **不要把 Obsidian 私人知识库直接公开。**
5. **不要让维护网站成为新的长期项目。**

如果某个设计决定让“发布 Markdown → push”变复杂，它默认就是错误方向。
