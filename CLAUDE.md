# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static single-origin website for **Marie's Little Library** — a non-profit (Bay Area, CA) sharing books, arts, and music with children ages 2–18. Built in Monty's Media format: self-contained HTML/CSS/JS files with no build step, no framework, no bundler. Open any `.html` directly in a browser.

Client contact: Marie Kay Valle — `Mkvglow@gmail.com` / (650) 438-2727

## File Map

| File | Role |
|---|---|
| `index.html` | Single-page homepage (hero, about, programs, gallery, donate, shop, contact) |
| `blog.html` | All-posts listing — category filter pills, live search, 6-per-page pagination |
| `blog-post.html` | Individual post template — reads `?id=` URL param, renders full post + sidebar |

All three files are fully self-contained (CSS + JS inline). No shared stylesheet or JS module exists by design (Monty's Media single-file format).

## Google Sheets Blog CMS

Blog content is driven by a public Google Sheet via the `gviz/tq` endpoint — **no API key required**.

**The same `SHEET_ID` constant must be kept in sync across all three files:**
```js
// index.html  — fetchBlogPosts()
// blog.html   — fetchPosts()
// blog-post.html — fetchPosts()
const SHEET_ID = 'YOUR_GOOGLE_SHEET_ID_HERE';
```

**Sheet columns (exact header names, case-sensitive):**
`id` · `title` · `date` (YYYY-MM-DD) · `category` · `excerpt` · `content` · `emoji` · `gradient` · `published` (TRUE/FALSE)

**Fetch pattern used in all three files:**
```js
const url  = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=out:json`;
const text = await (await fetch(url)).text();
const json = JSON.parse(text.replace(/^[^(]+\(/, '').replace(/\);\s*$/, ''));
const cols = json.table.cols.map(c => c.label.toLowerCase().trim());
// rows mapped to plain objects keyed by col name
```

When `SHEET_ID` is still the placeholder string, all three pages fall back to `FALLBACK_POSTS` (three hardcoded sample posts) so the site always renders.

## Design Tokens

All CSS is written with these custom properties (defined in `:root` in each file):

```css
--coral:  #F4845F   /* primary / CTAs */
--yellow: #F9C74F   /* accent */
--green:  #90BE6D   /* tertiary */
--cream:  #FFF8F0   /* page background */
--alt-bg: #FEF0E0   /* alternating section background */
--brown:  #3D2B1F   /* body text */
--brown-light: #7A5C47
--ff-head: 'Pacifico', cursive   /* headings */
--ff-body: 'Nunito', sans-serif  /* body */
```

Google Fonts loaded via CDN (`fonts.googleapis.com`). Only external dependency.

## PayPal Integration

All payment buttons post directly to `https://www.paypal.com/donate` or `https://www.paypal.com/cgi-bin/webscr`. The business email `Mkvglow@gmail.com` is hardcoded in every form's hidden `business` input. No server-side code — PayPal handles everything client-side.

## Key Patterns

- **Scroll reveal**: `.reveal` elements animate in via `IntersectionObserver`. After dynamically injecting HTML (e.g. blog cards), call `observeReveal()` on the new nodes.
- **Hamburger nav**: toggled via `classList.toggle('open')` on both `#nav-links` and `#ham`. Sub-page nav links point to `index.html#section`, not `#section`.
- **Lightbox** (index.html only): `data-index` on `.gallery-item` drives `openLightbox(idx)`. Keyboard: ESC closes, arrow keys navigate.
- **Blog routing**: `blog-post.html` reads `new URLSearchParams(window.location.search).get('id')` and matches against the fetched posts array.

## Deliverables in This Folder

- `Maries-Little-Library-Blog-Template.xlsx` — Google Sheets template (3 tabs: Posts, Category Options, Emoji & Gradient Guide). Upload to Google Drive and convert to Google Sheets format.
- `Maries-Little-Library-Blog-Guide.docx` — Non-technical CMS guide for Marie. Upload to Google Drive as a Google Doc.
- `Monty's Media — Client Website Intake (Responses)...csv` — Original client intake form response. Source of all copy and requirements.
