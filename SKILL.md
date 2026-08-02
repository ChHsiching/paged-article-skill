---
name: paged-article
description: Turn a link, file, or pasted text into a set of 3:4 vertical pages — a beautiful-article-style essay split into swipeable pages for Xiaohongshu and similar platforms. Each page is one chapter, rendered as standalone HTML with a real theme's design tokens (serif/sans, colors, SVG diagrams), screenshotted at 3x to PNG. Use when the user wants to make 小红书图文/分页文章/竖屏图文笔记 from a URL or content, mentions paged-article/分页文章, or asks to turn an article/post/video into shareable image pages without the card-illustration style of article-illustration.
---

Turn a **source** (URL, local file, pasted text, or image) into a set of **3:4 vertical pages** — a prose-first essay, split one-chapter-per-page, each rendered as standalone HTML with a real theme's design DNA, screenshotted at 3x to PNG. The agent does the writing, the layout, and the rendering. Output is ready to post: all page PNGs + a caption with title/body/hashtag options.

This is **beautiful-article's writing and design quality, delivered as paginated images**. The prose reads like an edited article (not card fragments or bullet dumps); the layout uses a real theme's color tokens, fonts, and structural discipline; SVG diagrams explain concepts where they earn their place. No AI-generated illustrations by default — visuals are hand-authored SVG and typography, zero image-API cost.

## What you produce

Every run produces all of these:

1. `NN.png` — the pages: **cover + content chapters + ending**, in order, each 3240×4320 (3x of 1080×1440)
2. `caption.md` — multiple title options (≤20 chars each, verified), multiple body options, hashtags, a sources block
3. `source.md` — the fetched/normalized source content + a translation note if the source isn't Chinese
4. `pages/NN.html` — the HTML source for each page (so the user can tweak and re-render)
5. `render.mjs` — the 3x Playwright render script

## Visual identity

- **Pages are 3:4 vertical** (1080×1440 CSS canvas, rendered at 3x = 3240×4320 PNG).
- **Prose-first**: body text is full subject-verb-object paragraphs with transition sentences, not fragments or bullet dumps. The article has a single restateable argument line a reader can follow page to page.
- **Theme-driven design**: each page uses a real theme's exact color tokens (hex values), font stacks, and structural rules — loaded via Google Fonts `<link>`, not local font files. The agent picks the theme with the user (see Step 3).
- **SVG diagrams, not AI illustrations**: when a concept earns a visual (a metaphor, a comparison, a process), the agent hand-writes an inline SVG using the theme's tokens. No image-generation API calls by default.
- **Structural emphasis over inline bolding**: key judgments go in an `aside` block (labeled callout); comparisons go in a `compare` grid; quotes go in a `quote` block. Body prose stays unbolded; `strong` is reserved for the one load-bearing phrase per paragraph.

## Where the outputs land — one folder per source

Default is `<cwd>/<source-slug>/`. Inside:

```
<source-slug>/
├── source.md           ← fetched/normalized source + translation note
├── pages/
│   ├── 01.html         ← cover
│   ├── 02.html         ← chapter 01
│   ├── ...
│   └── NN.html         ← ending
├── *.png                ← final 3:4 pages at 3x (01.png, 02.png, ...)
├── caption.md          ← title/body/hashtag options
└── render.mjs          ← 3x Playwright render script
```

## Environment check — resolve before generating

Before Step 1, confirm:

1. **playwright** resolves from the run directory: the render script uses `createRequire` to find it from cwd. Verify with `node -e "require('playwright')"` from the run dir. If missing, `npm i playwright`.
2. **Internet access** for Google Fonts: the render script's Playwright waits on `networkidle` + `document.fonts.ready`, so Newsreader / Hanken Grotesk / IBM Plex etc. load from Google Fonts. If offline, fonts fall back to system serif/sans — still readable, less polished.
3. **Source images** (if the user gave a screenshot/photo): the agent must be able to *see* the image to extract its content. If the image-analysis tool fails, ask the user to describe the image's text content verbatim — never skip it and guess.

## The pipeline

### Step 1 — Acquire the source content

The user gives a **link**, **local file**, **pasted text**, or **image**. Acquire by descending fallback:

1. Built-in web fetch / web read tools (fastest). Try this first.
2. For login-walled or JS-rendered pages, headless Playwright or browser MCP.
3. Ask the user to paste the content or point to a local file.

For **images** (screenshots, photos, diagrams): use an image-analysis tool to read the content. If the tool fails, **ask the user to transcribe the visible text** — images often carry load-bearing information (like a diagram's labels) that changes the whole article's accuracy. Never proceed on text alone if the source had an image you couldn't read.

Save normalized content to `source.md`. For non-Chinese sources, produce a **faithful, de-翻译腔 Chinese translation** — restructure to Chinese idiom, no word-for-word translation, no foreign word order or passive stacking. Keep the original text alongside (in a quote block or source note) for verbatim terms. Write the translation decisions into `source.md` (why a specific English phrase was translated a certain way).

**Done when** `source.md` exists and faithfully captures the source — **including any images**. If you skipped an image because you couldn't read it, you're not done.

### Step 2 — Read deeply, extract the argument line → page plan

Read `source.md`. Extract the **argument line** — the single thread a reader could restate in one sentence after finishing. This is an essay, not a card stack: pages flow into each other via transition sentences and content logic, not standalone points.

Identify the **cognitive anchors** — the core judgments, steps, structures, or turning points. Each anchor earns its own page (one chapter per page). Weight them as the source does: a one-line mention stays a one-line mention, not a full page.

Write the page plan to `source.md` (or a `shot-list.md`), one entry per page:
- page number, position (cover / chapter NN / ending), chapter name
- **core meaning** (one sentence)
- **what visual it needs** (SVG diagram? compare table? quote block? aside? or pure prose?)
- **structure type** (argument / before-after / process / layers / metaphor)

Then **STOP and get the user to confirm the page plan.** This gate is mandatory — a wrong skeleton wastes rendering time on every page.

**Done when** the page plan exists and the user has confirmed (or adjusted it).

### Step 3 — Theme selection: render multiple covers, let the user pick

The theme determines the entire visual DNA — colors, fonts, structural feel. **Don't pick for the user silently.** Instead:

1. **Recommend 3-5 themes** based on the content type (see `references/themes.md` for the full list with hex tokens and font stacks). State *why* each fits — e.g. "press: 衬线书卷气，适合观点/评论" / "freddie: 无衬线+明黄，亲和力强" / "shannon: 暗底工程感，吸睛".
2. **Render a cover thumbnail for each recommended theme** — same title, same lead, different theme tokens. This is zero-cost (pure HTML, no AI image generation). Use the `render.mjs` script at 1x for speed (thumbnails, not final).
3. **Show the user the thumbnails** and let them pick. Honor their choice even if it contradicts your recommendation.

Each theme is defined by its **exact CSS tokens** — background, foreground, accent, border, surface colors as hex values; font stacks (Google Fonts); weight limits; radius/shadow rules. The agent hardcodes these into each page's `<style>`. See `references/themes.md` for the token tables.

**Done when** the user picks a theme.

### Step 4 — Write and render all pages

The page plan and theme are confirmed. Now write each page as standalone HTML and render at 3x.

For each page, in order:

1. **Write the HTML** (`pages/NN.html`): a complete standalone document with the theme's tokens inlined in `<style>`, Google Fonts `<link>`, and the page's content. Use the structural components:
   - **Cover**: watermark (big low-opacity background text) + SVG hero visual + eyebrow tag + title + lead + meta row. The cover's visual hero follows **beautiful-article's cover design system** — see [`references/cover.md`](references/cover.md) for the 5 composition templates (A上字下图 / B大字盖图 / C分屏 / D拼贴 / E极简框) and the 5 hard constraints. The AI picks a composition template, invents a content-specific visual metaphor (the SVG), and layers the text — this is what produces the "出版物封面" quality, not a generic text-only cover.
   - **Chapter pages**: header (big chapter number + name + series) + h2 title + lead + prose paragraphs + SVG diagrams / compare tables / quote blocks / aside callouts as the content demands + footer (page number)
   - **Ending**: watermark + summary prose + note + END stamp
2. **Write the SVG diagrams** by hand, using the theme's color tokens. A diagram earns its place when it explains a concept faster than prose (a metaphor, a process, a comparison). Not every page needs one — pure prose is the default.
3. **Render at 3x**:
   ```bash
   node render.mjs pages/NN.html NN.png
   ```
4. **Verify** the rendered page: check for overflow (content past the 1440px canvas), overlap (text touching footer), font loading (serifs rendering, not fallback), and SVG integrity. Fix and re-render until clean.

The `render.mjs` script uses Playwright with `deviceScaleFactor: 3` — the CSS canvas stays 1080×1440, but the PNG is 3240×4320, giving serif strokes enough pixels to render sharply on mobile.

**Done when** every page exists as a 3x PNG and has no overflow, overlap, or rendering issues.

### Step 5 — Adjust loop

Show the user the full set. **Ask what to adjust.** Adjustments:

- **Text/layout** (HTML) — free, instant. Wording, emphasis, composition, SVG tweaks.
- **Theme switch** — re-render all pages with different tokens. Free but full re-render.

**Scope discipline**: every change stays inside the user's ask. A format report means fix the format — no rewriting sentences, no swapping metaphors, no adjusting unrelated pages.

**Done when** the user says the set is good or makes no more changes.

### Step 6 — Write the caption

Write `caption.md`:

- **Multiple title options** (≤20 chars each for Xiaohongshu), **different styles** (平实点题型 / 数字钩子型 / 痛点共鸣型). **None may duplicate the cover's title text.** Count characters with a script — agent eyeballing is unreliable, especially with English tokens (`spec` is 4 chars, `SDD` is 3).
- **Multiple body options**, each faithfully summarizing the content in plain text (no markdown — Xiaohongshu renders `**` and backticks literally).
- **A hashtag set.**
- **A sources block** listing every link the original source cited, for a pinned comment.

**Done when** `caption.md` exists with verified titles and multiple body options.

## Writing rules

These apply to every word on every page.

1. **Faithful to the source.** Only express what the source says. Weight follows the source.
2. **De-翻译腔.** For English sources: restructure to Chinese idiom, no word-for-word translation, no foreign word order, no passive stacking. Translate proper nouns correctly (查官方中译); keep original terms alongside (原文 prompt/code verbatim in a code/quote block).
3. **Prose-first, not card-fragments.** Body text is full paragraphs with transition sentences. The article has a single argument line. No standalone bullet-point cards.
4. **Structural emphasis.** Key judgments → `aside` callout. Comparisons → `compare` grid. Quotes → `quote` block. Body prose stays unbolded; `strong` for the one load-bearing phrase per paragraph max.
5. **No AI tone.** Write like an editor wrote it. No 首先/其次/综上所述, no 赋能/抓手/闭环, no 整理自/如图所示. Read each paragraph back — if it sounds like AI, rewrite.
6. **SVG diagrams earn their place.** Only when a concept is faster shown than told. Hand-authored, theme-token-colored, never decorative.

## Platform notes

- Images are 3:4 vertical (3240×4320 at 3x). Works for Xiaohongshu and similar.
- For Xiaohongshu: titles ≤20 chars, different from the cover text.
