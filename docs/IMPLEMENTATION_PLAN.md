# Implementation Plan — as-folio → Personal Intellectual Home

Maps P01–P07 of `PERSONAL_SITE_SPEC_v1.md` onto the current repository.
P00 established this baseline; a pre-P01 review then locked the architecture decisions in §2.
Neither P00 nor the pre-P01 correction touched UI, schema, dependencies, or demo content.

---

## 1. Baseline (P00)

Environment: Node `v24.19.0`; Yarn `4.13.0` as a standalone `yarn` on `PATH` via a user-level
Corepack shim (`/Users/wahrfreiheit/.local/bin/yarn` → the Corepack `dist/yarn.js`, version pinned
by `packageManager: yarn@4.13.0` in `package.json`); branch `main`; no `.env` (deployment falls
back to `https://example.github.io`, base `''`).

| Command | Result |
| --- | --- |
| `yarn --version` | `4.13.0` |
| `node --version` | `v24.19.0` |
| `yarn lint:ci` (`eslint . --quiet`) | PASS — 0 issues, exit 0 |
| `yarn test` (Vitest) | PASS — 6 files, 80 tests, exit 0 |
| `yarn test:types` (`astro check && tsc --noEmit`) | PASS — 0 errors, 0 warnings, 329 hints, exit 0 |
| `yarn build` (`astro build && pagefind`) | PASS — 110 pages in ~9.3s; Pagefind indexed 109 pages / 3110 words, exit 0 |
| `yarn test:e2e` (Playwright against `yarn preview`) | PASS — 9/9 chromium tests, 0 failures, exit 0 |

Notes:

- E2E now runs end-to-end: `playwright.config.ts` spawns `yarn preview` on `:4321`, which resolves
  because standalone `yarn` is on `PATH`. The previous P00 gap is closed.
- `yarn build` runs `prebuild` → `scripts/update-citations.ts` (OpenAlex). It completed without
  modifying `src/data/citations.yml`; no tracked file changed (`git status` shows only the untracked
  `PERSONAL_SITE_SPEC_v1.md`, `PROFILE_INPUT.md`, `avatar.jpg`, `docs/`, `tree.txt`).
- Pagefind logs 1 page without `<html>`: `/projects/astro-framework-external/` — an external
  redirect stub (`redirect` frontmatter). Expected, not a failure.
- `yarn lint` is `eslint . --fix`; the baseline uses `lint:ci` so no files are rewritten.
- The type check reports 329 hints (deprecated `z`, vendored Distill bundles, Astro script hints),
  unchanged from P00. Hints are not failures and are not part of the acceptance gate.

**No dependency changes were made. `yarn.lock` untouched.**

---

## 2. Pre-P01 architecture decisions (locked)

These were accepted in external review of P00 and are binding for P01–P07. Where a decision
conflicts with `PERSONAL_SITE_SPEC_v1.md`, the decision wins and the conflicting spec section is
named.

### D1 — The Yarn/E2E baseline is real

Recorded in §1. `yarn`, `yarn lint:ci`, `yarn test`, `yarn test:types`, `yarn build`, and
`yarn test:e2e` all pass on `main` at `6a30d3e`. No baseline gap remains.

### D2 — Zod is the single content-schema source of truth

`src/content.config.ts` is the only authoritative content schema. No phase may author or maintain a
parallel JSON schema — explicitly **not** `src/schemas/post.json`.

The upstream template ships `src/schemas/{post,project,people,teaching,book,announcement}.json` as
editor-hint artifacts, but nothing in this repo consumes them: frontmatter carries no `$schema`
key, there is no YAML language-server config, and a repo-wide search finds no other reference.
They would drift on the first schema edit, so they are not part of the schema story: P01 deletes
them unless a real consumer is demonstrated first (AGENTS.md: delete unused code outright).
No planned work targets `src/schemas/`.

### D3 — Writing frontmatter: one category, explicit layout

Writing posts move from the loose multi-category model to exactly one primary category, and from the
`distill: boolean` flag to an explicit layout enum:

```ts
category: z.enum([
  'life',
  'research',
  'history',
  'philosophy',
  'society',
]),
layout: z.enum(['essay', 'distill']).default('essay'),
deck: z.string().optional(),
lang: z.string().optional(),
comments: z.boolean().optional().default(false),
featured: z.boolean().optional().default(false),
series: z.object({ name: z.string(), order: z.number() }).optional(),
```

- `categories: string[]` is removed in P01 and every consumer migrates to `category` — no dual
  write, no alias. `category` is required for every post; existing demo posts get a mechanical
  `category:` backfill in P01 so the build stays green (demo purge remains P07).
- `layout` becomes the authoritative layout switch. `src/pages/writing/[slug].astro` selects
  `Essay`/`Post` vs `Distill` from `layout`; the `distill` boolean is removed in the same phase
  that introduces `layout` — one source, no fallback path.
- `lang` sets `<html lang>` (falling back to `site.lang`) and is the hook for CJK typography.
- `comments` gates Giscus per post on top of the global `site.giscus.enabled`.
- Everything else stays: `math`, `toc`, `relatedPosts`, `pinned`, `hidden`, `draft`, `robots`,
  `lastmod`, `redirect`, `image`, `distillAuthors`, `bibliography`, `citation_key`, and the per-post
  CDN flags `mermaid`, `chart_js`, `echarts`, `vega`, `plotly`, `pseudocode`, `typograms`,
  `tikzjax`, `map`, `img_comparison`, `code_diff`, `gallery`, `disqus` — all preserved, all still
  schema-validated.

### D4 — Filesystem directories are not category semantics

Writing content stays flat and structurally simple:

```text
src/content/posts/
├── some-history-essay.md
├── mixed-language-notes.md
└── distill-technical-writeup.mdx
```

The authoritative category is frontmatter (`category: history`). `src/content/posts/history/` is
**not** required and must not be created for category meaning — this overrides spec §23's
per-category folders. Public URLs are `/writing/<slug>/`, derived from the file slug alone, so
re-categorising a post never changes its URL: no redirect, no canonical/OG/sitemap churn. Category
archives live at `/writing/category/<category>/` and are computed from frontmatter.

### D5 — `/publications/` and `/cv/` stay top-level, stable routes

```text
/research/       narrative research hub
/publications/   full publication system (unchanged path)
/cv/             CV (unchanged path)
```

Do **not** move them to `/research/publications/` or `/research/cv/` (this overrides spec §2 and the
earlier P01 draft). The as-folio publication and CV pipelines already publish at these paths;
relocating them would create canonical, OG, sitemap, test, and inbound-link churn with no
user-facing benefit. `/research/` links internally to both.

Top-level navigation remains exactly:

```text
Home   Writing   Research   Projects   Notes   About
```

### D6 — Shared technical article widget layer (P04)

`src/components/writing/ArticleWidgets.astro` (or an equivalent name consistent with the codebase)
centralizes the conditional article-only enhancements that today live inline in
`src/layouts/Post.astro`: Mermaid, ECharts, Plotly, Vega/Vega-Lite, pseudocode, TikzJax, Leaflet,
gallery/PhotoSwipe, plus the comparable existing widgets (Chart.js, Typograms, img-comparison,
diff2html) and the code-copy / medium-zoom behaviour. All are frontmatter-controlled.

Both article layouts consume the same component:

```text
Essay.astro (from Post.astro) ─┐
                                ├─→ ArticleWidgets.astro
Distill.astro ──────────────────┘
```

`Distill.astro` is **not** assumed to provide these widgets: today it only emits the three
`distillpub` scripts (plus code-copy), so Distill posts would silently lose Mermaid/Plotly/etc.
The widget-loading implementation must not be duplicated between layouts. This is a P04
architecture requirement; extraction happens in P04, not pre-P01.

### D7 — Keep upstream component names

`Navbar.astro` and `Footer.astro` keep their names. Do not rename them to idealized
`SiteHeader`/`SiteFooter`. Prefer minimal compatible extension over cosmetic churn. Any earlier
`SiteHeader`/`SiteFooter` wording in this plan is corrected in §7.

### D8 — The serif font is not a solved dependency

`node_modules/@fontsource/` currently contains only `roboto`, `roboto-condensed`, and
`source-code-pro`. Source Serif 4 is **not** covered by existing dependencies; the spec's serif
stack names it only as the head of a fallback chain.

P02 must choose explicitly:

- **A.** add one narrowly-scoped dependency, `@fontsource/source-serif-4`, after review (self-hosted,
  Latin subset only); or
- **B.** use the system serif stack for V1 and defer any webfont.

CJK remains system-font in V1 either way (spec §17). No font package is added pre-P01.

### D9 — Multilingual/CJK fixtures are part of P01

P01 seeds throwaway fixtures sufficient to expose CJK and mixed-script typography problems **before**
P07 replaces them with real content:

| Fixture | Exposes |
| --- | --- |
| Chinese long-form essay | CJK line-height, punctuation, wrapping, `lang: zh-CN` |
| English long-form essay | serif Latin baseline, paragraph rhythm |
| Mixed Chinese/English essay | font fallback and punctuation transitions |
| Distill technical article | widgets, equations, citations, wide figures |
| Note example | Notes collection, route, minimal layout |
| Project example | Projects route/schema, card + detail |

They live flat under `src/content/posts/` (plus one `src/content/notes/` and one
`src/content/projects/` fixture) per D4, and are test/demo fixtures only — not final copy.

---

## 3. Architecture map

```text
as-folio (Astro 6, Tailwind v4, TS strict)
├── src/config/site.ts           single typed source of truth (identity, nav, features, labels)
├── src/content.config.ts        Zod schemas for 6 collections via glob loader  ← only schema source (D2)
├── src/content/
│   ├── posts/                   blog posts (.md/.mdx) — flat, category from frontmatter (D4) — feeds /blog/*
│   ├── projects/                project pages — feeds /projects/*
│   ├── people/                  lab members — /people/
│   ├── teaching/                courses — /teaching/
│   ├── announcements/           /news/ (also on homepage)
│   └── books/                   reading shelf — /books/
├── src/data/
│   ├── papers.bib               BibTeX source for publications (parsed at build time)
│   ├── coauthors.yml            LastName → { url, scholar, orcid }
│   ├── citations.yml            citation counts keyed by google_scholar_id (auto-updated)
│   ├── cv.yml / resume.json     RenderCV + JSONResume CV sources
│   ├── repositories.yml         GitHub users/repos for /repositories/
│   ├── venues.yml               venue abbreviation → full name
│   ├── distill-demo.bib         bibliography for the Distill demo post
│   └── about.mdx                bio fragment imported by the homepage
├── src/layouts/
│   ├── Base.astro               <head>, meta/OG, JSON-LD, analytics, dark-mode inline script, ClientRouter
│   ├── Page.astro               Base + centred .page-container
│   ├── Post.astro               blog layout: progress bar, TOC sidebar, related, share, Giscus, CDN widgets
│   └── Distill.astro            Distill-style academic layout (footnotes, citations, figures)
├── src/pages/
│   ├── index.astro              about/home: profile, announcements, latest posts, selected papers
│   ├── blog/                    index, [slug], tags, tag/[tag], category/[category], year/[year], page/[page]
│   ├── publications/            index (BibSearch), [key] (detail + citation meta)  ← path stays (D5)
│   ├── projects/                index (cards), [slug] (story + related publications)
│   ├── books.astro              reading shelf (Open Library covers)
│   ├── cv.astro                 RenderCV/JSONResume renderer + PDF download  ← path stays (D5)
│   ├── people/, teaching/       listings
│   ├── news.astro               announcements list
│   ├── repositories.astro       github-readme-stats cards from repositories.yml
│   ├── rss.xml.ts               RSS from posts
│   ├── sitemap.xml.ts           custom sitemap with git-based lastmod
│   ├── robots.txt.ts, 404.astro
│   └── og/[...path].png.ts      Satori → PNG OG cards for posts and projects
├── src/components/              Navbar, Footer, ThemeToggle, TOC, Giscus, SocialLinks, Figure, Tabs,
│                                Video/Audio, JupyterNotebook, Pagination, ReadingProgress, Newsletter,
│                                blog/* (PostCard, PostMeta, RelatedPosts, SocialShare),
│                                publications/* (BibSearch.tsx, BadgeSet*, scholar/inspire badges),
│                                projects/ProjectCard, cv/*, search/SearchTrigger, about/ProfilePhoto
├── src/schemas/                 upstream editor-hint JSON — no in-repo consumer; deleted in P01 (D2)
├── src/utils/                   bibtex.ts, authors.ts, posts.ts, cv.ts, date.ts, reading-time.ts,
│                                external-posts.ts, base.ts (+ colocated *.test.ts)
├── src/styles/                  global.css (Tailwind + fonts + component styles), _colors.css, _typography.css
├── scripts/update-citations.ts  OpenAlex DOI → citations.yml (prebuild + weekly workflow)
├── e2e/smoke.spec.ts            Playwright smoke tests (couples to "Albert Einstein" title)
└── .github/workflows/           ci.yml, deploy.yml (GitHub Pages), update-citations.yml, release-please.yml
```

Rendering model: `output: 'static'`, `trailingSlash: 'always'`, base-path aware via
`ASTRO_SITE` / `ASTRO_BASE` (env-first, consumed by `astro.config.mjs` → `site.ts`).
Client JS is limited to: React `BibSearch` (`client:load`), ninja-keys + Pagefind search,
theme toggle, reading progress, medium-zoom, card/GitHub-star script, Giscus.

---

## 4. as-folio features reusable unchanged

| Feature | Files | P00 verdict |
| --- | --- | --- |
| **BibTeX parsing** | `src/utils/bibtex.ts`, `src/utils/bibtex.test.ts` | Reuse unchanged. Internal-field stripping, `getCleanBibtex`, author/venue helpers all spec-agnostic. |
| **Publications plumbing** | `src/pages/publications/index.astro`, `[key].astro`, `src/components/publications/BibSearch.tsx`, `BadgeSet*`, `src/data/venues.yml`, `coauthors.yml` | Reuse at the existing `/publications/` path (D5). Only styling/badge-noise reduction in P05; `papers.bib` content replaced in P07. |
| **Citation updater** | `scripts/update-citations.ts`, `package.json` `prebuild`, `.github/workflows/update-citations.yml` | Reuse unchanged, except replace `POLITE_POOL_EMAIL = 'hello@example.com'` with a real address (P07). |
| **Pagefind search** | `package.json` build script, `src/components/search/SearchTrigger.astro`, `data-pagefind-body` in layouts | Reuse. P06 extends index coverage to Notes and verifies draft/hidden exclusion. |
| **Dark mode** | `src/components/ThemeToggle.astro`, `[data-theme]` blocks in `_colors.css`, inline FOUC script in `Base.astro` | Reuse mechanism unchanged; P02 only swaps token values. |
| **Distill layout** | `src/layouts/Distill.astro`, `src/content/posts/distill-style-post.mdx`, `src/data/distill-demo.bib` | Keep as the technical/scientific article layout, but it must be wired to the shared widget layer in P04 (D6) — it does not provide Post.astro's CDN widgets today. |
| **Projects plumbing** | `src/content.config.ts` `projects`, `src/pages/projects/index.astro`, `[slug].astro`, `src/components/projects/ProjectCard.astro` | Reuse plumbing; P05 restyles cards + detail layout. |
| **Books / reading** | `src/content.config.ts` `books` (status/stars/dates), `src/pages/books.astro` | Reuse; P01 routes it at `/reading/` (not in the 6-item nav), P05 reshapes into Reading Now / Recently Finished / Selected. |
| **Comments (Giscus)** | `src/components/Giscus.astro`, `site.giscus`, `Giscus.astro` lazy-load pattern | Reuse; P01 adds per-post `comments: boolean` (D3). |
| **RSS** | `src/pages/rss.xml.ts` | Reuse; P01 updates the post link path from `/blog/` to `/writing/`. |
| **Sitemap** | `src/pages/sitemap.xml.ts` | Reuse; P01/P06 update the static-route list (keeping `/publications/` and `/cv/` top-level) and add Notes. |
| **OG images** | `src/pages/og/[...path].png.ts` | Reuse; P06 extends to Writing routes / refresh accent color. |
| **CV** | `src/pages/cv.astro`, `src/utils/cv.ts`, `src/data/cv.yml`, `resume.json` | Reuse renderer at the existing `/cv/` path (D5); P07 replaces data. |
| **Navbar / Footer** | `src/components/Navbar.astro`, `src/components/Footer.astro` | Keep the upstream names and extend in place (D7); only `site.navbar.items` / `site.footer` config changes. |

---

## 5. Demo / persona content to remove or replace (P07)

**Identity & config**

- `src/config/site.ts` — Einstein title/description, `author.*` (name, email, avatar, subtitle, moreInfo),
  `socials.*` (scholar `qc6CJjYAAAAJ`, inspire `1010907`, `cv_pdf` `example_pdf.pdf`),
  `footer.text`, `navbar.items`, `blog.name/description/displayTags`, `pages.*.description`,
  `teaching.calendarId: 'test@gmail.com'`, `theme.color`.
  Real values come from `PROFILE_INPUT.md` (currently TODOs) — placeholders must be explicit until then.
- `src/data/about.mdx` — template boilerplate biography.
- `avatar.jpg` (repo root, untracked) — candidate real avatar to move into `public/assets/img/`.

**Bibliography / CV / repos**

- `src/data/papers.bib` — 9 Einstein entries; `src/data/citations.yml` — 9 matching keys.
- `src/data/coauthors.yml` — Einstein co-author links.
- `src/data/cv.yml` + `src/data/resume.json` — Einstein CV.
- `src/data/repositories.yml` — `torvalds`, `dadangnh/as-folio`, `git/git`, `python/cpython`, `astro/astro`, `tailwindlabs/tailwindcss`.
- `src/data/distill-demo.bib` — demo bibliography (keep only if a Distill post remains).

**Content collections**

- `src/content/posts/*` (23 files) — all demo posts; P01 backfills `category` on them and reuses 2–3
  as schema fixtures; the rest are replaced in P07.
- `src/content/projects/*` (12 files) — Einstein projects.
- `src/content/people/*` (6), `src/content/teaching/*` (4), `src/content/books/*` (10),
  `src/content/announcements/*` (8) — demo.

**Assets**

- `public/assets/img/prof_pic.{jpg,svg,png}` (incl. 13.7 MB `prof_pic_color.png`), `planck.jpg`,
  `curie.jpg`, `bohr.jpg`, `heisenberg.jpg`, `rhino.png`, numbered `1–12.jpg`,
  `book_covers/the_godfather.jpg`, `publication_preview/*.gif`, `as_folio_*.png`, `as_folio_cv.mhtml`.
- `public/assets/audio/chopin-nocturne.ogg` (12.9 MB), `bach-bwv543-fugue.ogg` (6.6 MB).
- `public/assets/pdf/` is empty (`.gitkeep`); `cv_pdf`/`pdfPath` point at a missing `example_pdf.pdf`.

**Template metadata**

- `README.md`, `QUICKSTART.md`, `CHANGELOG.md`, `package.json` name/author/homepage/repository,
  `release-please-config.json`, `netlify.toml`, `vercel.json`, `.gitlab-ci.yml`,
  `.github/workflows/deploy.yml` (`dadangnh` fallbacks).
- `e2e/smoke.spec.ts` asserts `toHaveTitle(/Albert Einstein/)` and visits `/blog/` — must be updated
  in the phase that renames the identity and the Writing route (P01 for `/writing/`, P07 for the
  name), never weakened to force a pass.

---

## 6. Phase mapping P01–P07 → current files

### P01 — Information Architecture (no visual redesign)

Target routes: `Home / Writing / Research / Projects / Notes / About`, plus stable
`/publications/` and `/cv/` (D5).

| Work | Files |
| --- | --- |
| Nav → 6 items (`Home Writing Research Projects Notes About`) | `src/config/site.ts` (`navbar.items`); optional new `src/config/navigation.ts` per spec §23 |
| Writing route rename + archives | move/rename `src/pages/blog/**` → `src/pages/writing/**` (`index.astro`, `[slug].astro`, `tags.astro`, `tag/[tag]/`, `category/[category]/`, `year/[year]/`, `page/[page]/`); URLs are `/writing/<slug>/` |
| Schema authority | `src/content.config.ts` only (D2). Delete the unconsumed `src/schemas/*.json` editor artifacts — no parallel JSON schema is authored or maintained. |
| Single category + layout enum (`deck`, `category`, `lang`, `layout`, `comments`, `featured`, `series`) | `src/content.config.ts` posts schema (remove `categories: string[]` and the `distill` boolean); mechanical `category:` backfill across existing demo posts; consumers: `src/pages/writing/*`, `src/layouts/Post.astro`, `Distill.astro`, `src/components/blog/PostMeta.astro`, `src/utils/posts.ts` |
| Posts stay flat (no per-category folders) | content remains `src/content/posts/*.md(x)`; category meaning comes only from frontmatter (D4) |
| Notes collection + routes | `src/content.config.ts` (new `notes`), new `src/content/notes/`, new `src/pages/notes/index.astro`, `src/pages/notes/[slug].astro` |
| Research hub | new `src/pages/research/index.astro` (narrative), linking internally to `/publications/` and `/cv/`; `src/pages/publications/**` and `src/pages/cv.astro` keep their current paths — no `/research/publications/`, no `/research/cv/` (D5) |
| Reading route | `src/pages/books.astro` → `src/pages/reading.astro` (homepage/About link only; not in the 6-item nav) |
| About route | new `src/pages/about.astro` (currently the about content lives in `src/pages/index.astro`); `src/data/about.mdx` |
| Remove/hide unused | `src/pages/teaching/index.astro`, `src/pages/people/index.astro`, `src/pages/repositories.astro`, `src/pages/news.astro`, `src/components/Newsletter.astro`, related nav entries in `site.ts` |
| RSS / sitemap / e2e paths | `src/pages/rss.xml.ts`, `src/pages/sitemap.xml.ts`, `e2e/smoke.spec.ts` (`/blog/` → `/writing/`) |
| Multilingual/CJK fixtures | flat fixtures per D9: Chinese long-form, English long-form, mixed zh/en, Distill technical, one Note, one Project |

### P02 — Design Tokens (no page logic change)

| Work | Files |
| --- | --- |
| Warm-paper light / restrained dark / teal accent | `src/styles/_colors.css` (existing `--global-*` variables + `html[data-theme='dark']`) |
| UI sans + long-form serif stacks, type scale | `src/styles/_typography.css`, `@theme` block; `global.css` `@font-face` subset. Serif is decision D8: either add `@fontsource/source-serif-4` (self-hosted, Latin subset) or ship the system serif stack for V1 — decide and record in CUSTOMIZE.md. CJK stays system-font. |
| Spacing / border / radius / article-width / motion tokens | `src/styles/global.css`, new `src/styles/_motion.css`, new `src/styles/_article.css` |
| `prefers-reduced-motion` | new `src/styles/_motion.css` |
| Accent override plumbing | `src/config/site.ts` (`theme.color`), consumed by `src/layouts/Base.astro` inline `--user-*` vars |
| Verification surface | temporary token test page or existing components in light/dark desktop/mobile |

### P03 — Homepage

| Work | Files |
| --- | --- |
| Hero, IdentityGrid, LatestWriting, SelectedResearch, SelectedProjects, NowReading | `src/pages/index.astro` + new `src/components/home/*.astro` per spec §18; data from `site.ts`, `src/content/*`, `src/data/papers.bib`, `src/content/books` |
| Section headers/tags/empty states | new `src/components/common/*` |
| Footer | `src/components/Footer.astro` (name unchanged, D7), `site.footer` (`position`, `text`) |
| JSON-LD `Person` | `src/layouts/Base.astro` (`isHome`) |
| Mobile stacking | `src/pages/index.astro` styles |

### P04 — Writing / Retypeset

| Work | Files |
| --- | --- |
| Writing index (All + 5 categories, year-grouped) | `src/pages/writing/index.astro`, `category/[category]/index.astro`, `year/[year]/index.astro`, `src/components/blog/PostCard.astro`, `PostMeta.astro` |
| Essay layout | `src/layouts/Post.astro` (or a dedicated `Essay.astro` per spec §19), `src/pages/writing/[slug].astro` |
| **Shared widget layer** | new `src/components/writing/ArticleWidgets.astro` (D6) — all conditional CDN/widget loading extracted from `Post.astro` (Mermaid, Chart.js, ECharts, Vega/Vega-Lite, Plotly, pseudocode, Typograms, TikzJax, Leaflet, img-comparison, diff2html, gallery, code-copy, medium-zoom); consumed by BOTH `Essay.astro`/`Post.astro` and `src/layouts/Distill.astro` — no duplicated loader |
| Article header/meta/TOC/footer/series/related/prev-next | new `src/components/writing/*`; `src/components/TOC.astro`, `src/components/blog/RelatedPosts.astro`, `src/utils/posts.ts` |
| Footnotes, blockquotes, figures, pull quotes, sidenotes | `src/styles/_article.css`, `src/components/Figure.astro`, new `PullQuote.astro` / `Sidenote.astro` |
| Giscus per post | `src/components/Giscus.astro`, `site.giscus`, `comments` frontmatter (D3) |
| Print CSS | new `src/styles/_print.css` |
| Distill preserved, wired to widgets | `src/layouts/Distill.astro`, `src/content/posts/distill-style-post.mdx` |
| Tests | `src/utils/posts.test.ts`, `src/utils/reading-time.test.ts` |

### P05 — Research / Projects / About

| Work | Files |
| --- | --- |
| Research narrative + selected research cards | `src/pages/research/index.astro`, new `src/components/research/ResearchCard.astro`; data from new content collection or `src/data/`; internal links to `/publications/` and `/cv/` |
| Selected + full publications | `src/components/research/PublicationList.astro`, `src/pages/publications/index.astro` (path unchanged, D5), `src/components/publications/BibSearch.tsx`, `BadgeSet*` |
| CV | `src/pages/cv.astro` (path unchanged, D5), `src/config/site.ts` (`cv.format`, `pdfPath`) |
| Project category groups + clean cards | `src/pages/projects/index.astro`, `src/components/projects/ProjectCard.astro` (categories from frontmatter `category`) |
| Project story detail | `src/pages/projects/[slug].astro`, new `src/components/projects/ProjectMeta.astro`, `related_publications` → `src/utils/bibtex.ts` |
| About narrative-first | `src/pages/about.astro`, `src/data/about.mdx`, `src/components/about/ProfilePhoto.astro`, `src/components/SocialLinks.astro` |

### P06 — Search / SEO / Performance

| Work | Files |
| --- | --- |
| Pagefind coverage incl. Notes, drafts/hidden excluded | `package.json` build script, `data-pagefind-body` in `src/layouts/Base.astro` / `Post.astro` / new `Essay.astro`, `src/components/search/SearchTrigger.astro`, `src/pages/rss.xml.ts` filter |
| Canonical / OG / Twitter / JSON-LD | `src/layouts/Base.astro`, `src/layouts/Post.astro` (BlogPosting + BreadcrumbList), `src/pages/og/[...path].png.ts` |
| RSS / sitemap routes | `src/pages/rss.xml.ts`, `src/pages/sitemap.xml.ts`, `src/pages/robots.txt.ts`; `/publications/` and `/cv/` remain top-level entries |
| Accessibility (focus, keyboard, contrast, reduced motion) | `src/styles/global.css`, `_colors.css`, `_motion.css`, `src/components/Navbar.astro`, `SearchTrigger.astro`, `ThemeToggle.astro`, `src/pages/404.astro` |
| Image dimensions / optimization | `astro.config.mjs` (`image.domains`), `src/components/Figure.astro`, `src/pages/**` usage |
| Client JS audit | `src/components/publications/BibSearch.tsx`, `src/pages/projects/index.astro` script, `src/components/search/SearchTrigger.astro` |

### P07 — Content & Launch

| Work | Files |
| --- | --- |
| Replace identity | `src/config/site.ts`, `src/data/about.mdx`, `avatar.jpg` → `public/assets/img/` |
| Replace publications | `src/data/papers.bib`, `src/data/citations.yml`, `src/data/coauthors.yml`, `src/data/venues.yml` |
| Replace CV | `src/data/cv.yml`, `src/data/resume.json`, `public/assets/pdf/` |
| First Writing posts / Projects / Notes (replace P01 fixtures) | `src/content/posts/`, `src/content/projects/`, `src/content/notes/` |
| Remove demo assets | `public/assets/img/*`, `public/assets/audio/*` (see §5) |
| Repos page decision | `src/data/repositories.yml`, `src/pages/repositories.astro` |
| Giscus config | `src/config/site.ts` (`giscus.*`) |
| Deployment | `.env.example`, `astro.config.mjs`, `.github/workflows/deploy.yml`, `netlify.toml`/`vercel.json` |
| Test coupling | `e2e/smoke.spec.ts` (persona title), `src/utils/*.test.ts` fixtures |
| Template docs | `README.md`, `QUICKSTART.md`, `CUSTOMIZE.md`, `CHANGELOG.md`, `package.json` metadata |
| Launch checklist | new `docs/LAUNCH_CHECKLIST.md` |

---

## 7. Constraints carried into later phases

- Follow `/AGENTS.md` + `/CLAUDE.md`: yarn only (never `npm`/`npx`), `yarn add -D` (never
  `--dev`), no hardcoded persona strings in components, Zod schema for every new field,
  `yarn build` must exit 0, `data-theme` (never Tailwind `dark:`), no `innerHTML` with untrusted
  content.
- **Schema:** `src/content.config.ts` is the only content schema; no duplicated JSON schema
  (D2/D3).
- **Writing URLs:** `/writing/<slug>/` derives from the file slug only; category never moves a
  file or a URL (D4).
- **Routes:** `/research/` is a hub; `/publications/` and `/cv/` stay top-level and stable (D5).
- **Component names:** keep upstream `Navbar.astro` / `Footer.astro`; extend, don't rename (D7).
- **Widgets:** both article layouts load enhancements through one shared component (D6).
- **Fonts:** existing Fontsource packages cover Roboto/Roboto Condensed/Source Code Pro only.
  The long-form serif requires an explicit P02 decision — one narrow `@fontsource/source-serif-4`
  dependency, or the system serif stack for V1 (D8). A CJK serif webfont is deliberately deferred
  (spec §17 V1).
- Search/OG/sitemap are base-path aware (`src/utils/base.ts`, `ASTRO_BASE`); route renames must
  update every hardcoded `/blog/` reference (`src/pages/rss.xml.ts`, `src/layouts/Post.astro`
  breadcrumbs, `e2e/smoke.spec.ts`) in the same phase, without weakening tests to pass.
