# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static single-origin website for **Marie's Little Library** — a non-profit (Bay Area, CA) sharing books, arts, and music with children ages 2–18. Built in Monty's Media format: self-contained HTML/CSS/JS files with no build step, no framework, no bundler. Open any `.html` directly in a browser.

Client: Marie Kay Valle — `Mkvglow@gmail.com` / (650) 438-2727 / prefers Text/SMS

## Deployment

- **GitHub repo** → connected to **Vercel**, auto-deploys on every push
- **Live preview URL**: https://maries-little-library.vercel.app/
- **Custom domain**: not yet purchased — everything works on the `.vercel.app` URL in the meantime
- `vercel.json` enables `cleanUrls` (strips `.html` from URLs) and adds security headers. Existing `href="blog.html"` links still work — Vercel redirects them transparently.

## File Map

| File | Role |
|---|---|
| `index.html` | Single-page homepage (hero, about, programs, gallery, donate, shop, contact) |
| `blog.html` | All-posts listing — category filter pills, live search, 6-per-page pagination |
| `blog-post.html` | Individual post template — reads `?id=` URL param, renders full post + sidebar |
| `vercel.json` | Clean URLs + security headers |

All three HTML files are fully self-contained (CSS + JS inline). No shared stylesheet or JS module exists by design (Monty's Media single-file format).

## Outstanding Config Items (Monty's Media to action)

These are placeholders waiting on client assets or third-party setup:

| What | File(s) | Action |
|---|---|---|
| **Formspree ID** | `index.html` | Swap `action="mailto:Mkvglow@gmail.com"` on the contact form with `action="https://formspree.io/f/XXXX"` |
| ~~**Sheet ID**~~ | ~~All 3 HTML files~~ | ✅ Done — `13Ncs-PLCjw4-zJvOT0O4JdtejPdrlCFYclBvOT3DBGw` set in all 3 files. Sheet must be published to web (File → Share → Publish to web) before going live. |
| **Logo file** | All 3 HTML files | Replace `.nav-logo` text in nav + footer with `<img>` tag |
| **Gallery photos** | `index.html` | Replace the 9 `.ph-gallery` emoji placeholder divs with `<img>` tags |
| **Real address** | `index.html` | Update Google Maps iframe `src` — use Maps Embed API with a free API key for reliability |
| **Social links** | All 3 HTML files | Fill the 3 footer `href="#"` placeholders (Facebook, Instagram, TikTok) |
| **Custom domain** | Vercel dashboard | Point domain → Vercel once client purchases one (~$12/yr) |

## Outstanding Client Actions (Marie to complete)

- Sign up at formspree.io → create form → send Formspree ID
- Confirm PayPal Business account connected to `Mkvglow@gmail.com`
- ~~Follow Blog Guide Steps 1A–1C → publish Google Sheet → send Sheet ID~~ ✅ AJ handles sheet setup — Marie just edits the shared sheet
- Send logo file (PNG, transparent background preferred)
- Send gallery photos (8–9 images)
- Send real address for map
- Send Facebook, Instagram, TikTok profile URLs
- Buy domain when ready

## Google Sheets Blog CMS

Blog content is driven by a public Google Sheet via the `gviz/tq` endpoint — no API key required.

**The same `SHEET_ID` constant must be kept in sync across all three files:**
```js
// index.html  — fetchBlogPosts()
// blog.html   — fetchPosts()
// blog-post.html — fetchPosts()
const SHEET_ID = '13Ncs-PLCjw4-zJvOT0O4JdtejPdrlCFYclBvOT3DBGw'; // ✅ Set — Marie's shared sheet
```

**Sheet columns (exact header names, case-sensitive):**
`id` · `title` · `date` (YYYY-MM-DD) · `category` · `excerpt` · `content` · `emoji` · `gradient` · `image_url` · `published` (TRUE/FALSE)

- `image_url` is optional — if empty or the image fails to load, falls back to `gradient` + `emoji` automatically
- `published = FALSE` saves as draft, hidden from the site

**Fetch pattern used in all three files:**
```js
const url  = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=out:json`;
const text = await (await fetch(url)).text();
const json = JSON.parse(text.replace(/^[^(]+\(/, '').replace(/\);\s*$/, ''));
const cols = json.table.cols.map(c => c.label.toLowerCase().trim());
// rows mapped to plain objects keyed by col name
```

When `SHEET_ID` is still the placeholder string, all three pages fall back to `FALLBACK_POSTS` (three hardcoded sample posts) so the site always renders.

## Photo Fallback Pattern

All three files use a `postThumb()` / `heroBg` pattern: if `post.image_url` is present, it sets a CSS `background:url(...)`. An `Image.onerror` handler fires if the URL fails and swaps back to gradient + emoji. No layout ever breaks.

```js
const photo = post.image_url && post.image_url.trim();
// if photo → background:url('...') center/cover no-repeat
// if no photo → background: gradient + emoji centered
```

Google Drive direct image URL format: `drive.google.com/uc?export=view&id=FILE_ID`

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
- **Clean URLs**: `vercel.json` strips `.html` so `/blog.html` → `/blog` in production. Local file:// testing still uses `.html` links which is fine.

## Deliverables in This Folder

- `Maries-Little-Library-Blog-Template.xlsx` — Google Sheets template (3 tabs: Posts, Category Options, Emoji & Gradient Guide). Share with Marie via Google Drive — AJ owns and manages setup; Marie edits posts directly.
- `Maries-Little-Library-Blog-Guide.docx` — Non-technical CMS guide for Marie. Share via Google Drive. Step 1 (sheet setup/publish/ID) is handled by AJ — Marie starts from Step 2.
- `Monty's Media — Client Website Intake (Responses)...csv` — Original client intake form response. Source of all copy and requirements.
