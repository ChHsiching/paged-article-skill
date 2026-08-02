# paged-article

Turn a link, file, or pasted text into a set of 3:4 vertical pages — a beautiful-article-style essay split into swipeable pages for Xiaohongshu.

## What it does

Give it a URL, a local file, pasted text, or an image. It produces:

- **3:4 pages** (3240×4320 PNG at 3x) — cover + one chapter per page + ending
- **caption.md** — multiple title/body/hashtag options for posting
- **source.md** — the normalized source + translation notes
- **pages/*.html** — the HTML source for each page (re-renderable)

The prose reads like an edited article (not card fragments). The layout uses a real theme's design tokens. SVG diagrams explain concepts where they earn their place. Zero AI-image cost by default.

## Flow

1. **Acquire** the source (URL/file/text/image) → `source.md`
2. **Plan** the chapters (AI extracts the argument line, user confirms)
3. **Pick a theme** (render multiple cover thumbnails, user chooses)
4. **Write + render** all pages at 3x
5. **Adjust** (free HTML tweaks until the user is happy)
6. **Caption** (titles ≤20 chars, body options, hashtags, sources)

## Install

Copy `SKILL.md`, `render.mjs`, and `references/` into your skills directory. Requires `playwright` (`npm i playwright`) in the run directory.

## Files

```
paged-article-skill/
├── SKILL.md              ← the skill (steps + rules)
├── render.mjs            ← 3x Playwright render script
└── references/
    └── themes.md         ← theme token tables (hex + fonts + Google Fonts links)
```
