# AKTU Hub — Build Context & Spec

> Reference doc for rebuilding this project from scratch. Paste this whole file to Claude at the start of a future session instead of re-explaining the project.

**Live site:** https://aktuverse.vercel.app/
**Deployment:** Vercel, auto-deploy from GitHub `main` branch
**Architecture:** Single HTML file. No build step, no framework, no backend.

---

## 1. What this project is

A toolkit site for AKTU (Dr. A.P.J. Abdul Kalam Technical University) students — academic calculators (CGPA, attendance, bunk planning, percentage) plus quick links to AKTU's official portals. Built and maintained by Frank (B.Tech CSE, UCER Prayagraj).

SEO/meta branding name: **"Academic Tools"**
On-page logo/nav branding: **"AKTU Hub"**
(Both are intentional — keep this split when rebuilding: meta tags say "Academic Tools" for generic search reach, UI says "AKTU Hub" for the actual product identity.)

## 2. Hard constraints (don't violate these)

- **Single file.** Everything — HTML, CSS, JS — lives in one `.html` file. No separate `.css`/`.js` files, no bundler, no npm.
- **No frameworks.** Vanilla JS only. No React/Vue/jQuery.
- **No backend, no database.** All logic is client-side. No localStorage currently used for calculator inputs (values reset on reload — this is current behavior, not a bug).
- **Page-swap navigation**, not multi-page routing. All "pages" are `<div class="page">` blocks toggled via `showPage(id)`, with `history.pushState` + `#hash` for deep linking and back/forward support.
- Prefer **targeted edits** (find-and-replace specific blocks) over full-file rewrites when iterating.

## 3. Tech stack

| Layer | Choice |
|---|---|
| Markup | Plain HTML5 |
| Styling | Plain CSS3, CSS custom properties (`:root` variables) — no Sass/Tailwind/etc. |
| Font | Inter (Google Fonts, weights 400/500/600/700/800) |
| Interactivity | Vanilla JS, no libraries |
| Icons | Inline SVG (stroke-based, 14–18px, `stroke-width="2"`) |
| Analytics | Google Analytics (gtag.js) + Microsoft Clarity |
| SEO | Meta tags, Open Graph, Twitter Card, JSON-LD `WebSite` schema, hidden `.seo-text` block for crawlers |
| Hosting | Vercel, deployed from GitHub |

## 4. Design system (CSS variables — reuse these exactly)

```css
:root{
  --green:#1C3D2B;        /* primary text/brand color */
  --green-mid:#2E6B47;    /* accents, links, "ok" states */
  --green-light:#EAF3DE;  /* badge backgrounds, icon bg */
  --green-faint:#F0F5E9;
  --bg:#F2EFE8;            /* page background (warm off-white) */
  --surface:#FFFFFF;       /* cards */
  --surface2:#F7F5EE;      /* input backgrounds */
  --muted:rgba(28,61,43,.42);
  --border:rgba(28,61,43,.16);
  --border-h:rgba(28,61,43,.32); /* hover border */
  --r:14px;                /* card border-radius */
  --r-sm:10px;
  --err:#B83232;
  --err-bg:#FEF0EE;
  --err-border:rgba(184,50,50,.20);
}
```

**Visual language:** warm off-white background, deep green as the only brand hue, soft 1px borders, subtle shadows on hover (`box-shadow:0 2px 12px rgba(28,61,43,.07)`), no harsh black, no gradients, no drop shadows on text. Rounded corners everywhere (10–14px). Tool icons get their own pastel tint per category (e.g. blue for CGPA, orange for bunk, purple for percentage) — see section 6.

**Type scale:** Hero h1 `clamp(26px,6vw,40px)` weight 700, tool page h2 `22px` weight 700, body/labels `11–15px`. Letter-spacing slightly negative on headings (`-.02em` to `-.04em`), slightly positive on uppercase labels (`.07em`–`.1em`).

## 5. Page inventory

All pages live in one file as sibling `<div id="page-{id}" class="page">` blocks. `home` starts with class `active`.

| id | Title | Purpose |
|---|---|---|
| `home` | — | Hero + tools grid + resources + important links + footer |
| `attendance` | Attendance Calculator | How many more classes needed to hit target % |
| `bunk` | Bunk Predictor | How many classes can be safely skipped |
| `cgpa` | CGPA Calculator | Simple average of SGPAs |
| `cgpa_cb` | Credit-Based CGPA | SGPA × credits weighted average |
| `percentage` | Percentage Calculator | Total marks scored / total marks |

Routing: `showPage(id)` hides all `.page`, shows `#page-{id}`, syncs nav active states, scrolls to top, lazy-inits semester-list tools on first visit, and does `history.pushState`. `popstate` and initial `location.hash` are both handled so deep links and back/forward work.

## 6. Home page content (verbatim, for rebuilding copy)

**Hero:**
- H1: "Your Academic toolkit" / em: "Nothing extra."
- Subline: "CGPA · Attendance · Percentage"
- Hidden SEO paragraph (for crawlers, not visible): mentions CGPA/SGPA/Credit-Based CGPA/Attendance/Attendance Recovery/Bunk/Percentage calculators and AKTU students explicitly.

**Tools grid** (5 cards, icon bg color + icon color per card):
1. Attendance Calculator — bg `#EAF3DE` / icon `#2E6B47` — "How many classes you need to attend"
2. CGPA Calculator — bg `#DEE9F3` / icon `#1C5C8A` — "Simple average of your SGPAs"
3. Credit-Based CGPA — bg `#E8EEF8` / icon `#2C4C8A` — "Weighted by semester credits"
4. Bunk Predictor — bg `#F3E8DE` / icon `#7A3B1E` — "Safe bunk planning"
5. Percentage Calculator — bg `#EDE8F3` / icon `#5B3A7A` — "Overall marks percentage"

**Resources section:** "Notes" and "PYQs" cards — both currently trigger a `comingSoon(feature)` toast instead of navigating (feature not built yet).

**Important Links section** (external, `target="_blank"`):
- ERP → `https://erp.aktu.ac.in`
- One View → `https://oneview.aktu.ac.in`
- Website → `https://aktu.ac.in`
- Syllabus → `https://aktu.ac.in/syllabus.php`

**Footer:** "Built for students · Academic Tools"

## 7. Calculator logic (exact formulas — reuse, don't reinvent)

**Attendance % helper:** `pct(a,t) = round(a/t*1000)/10` (1 decimal place)

**Attendance Calculator** (`calcAttendance`)
- If current % ≥ target: can bunk = `floor((attended - target/100*total) / (target/100))`
- If current % < target: need to attend = `ceil((target*total/100 - attended) / (1 - target/100))` more **consecutive** lectures
- Validates: target 1–100, attended ≥ 0, total > 0, attended ≤ total

**Bunk Predictor** (`calcBunk`) — same underlying math as Attendance, framed oppositely:
- If current < target → "cannot bunk", shows classes needed to reach target
- If current ≥ target → "can bunk N more", same `floor` formula as above

**CGPA Calculator** (`calcCgpa`) — simple mean: `sum(SGPA) / count`, 2 decimal places. SGPA validated 0–10. 1–8 semester rows, default 4, add/remove dynamically.

**Credit-Based CGPA** (`calcCb`) — weighted mean: `sum(SGPA × credits) / sum(credits)`, 2 decimals. SGPA 0–10, credits 1–100. Also displays semester count + total credits as stat blocks.

**Percentage Calculator** (`calcPct`) — `sum(scored) / sum(total) * 100`, 2 decimals. Validates scored ≥ 0, total > 0, scored ≤ total per row.

All four multi-row tools (CGPA, Credit CGPA, Percentage) share the same UI pattern: dynamically added "Semester N" cards with a delete (×) button, min 1 / max 8 rows, inline per-field error messages, and a banner-level error (`.err-banner`) for row-level validation failures.

## 8. JS architecture notes

- Global functions, no modules/IIFE wrapping — kept flat for easy single-file editing.
- Each multi-row tool (`cgpa`, `cb`, `pct`) follows the identical pattern: `initX()` resets and seeds 4 rows → `appendXSem()` creates a row with a unique incrementing `uid` → `relabelX()` renumbers visible labels after add/delete → `addXSem()`/`delXSem(uid)` enforce 1–8 row bounds → `calcX()` validates and computes.
- Shared helpers: `step(id,delta)` for +/- steppers, `setErr`/`showBanner`/`clearErrs` for validation UI, `pct(a,t)` for percentage math.
- Enter key triggers the relevant page's calculate function via a single global `keydown` listener that checks `.page.active`'s id.
- Mobile nav: hamburger toggles `.nav-mob.open`, closes on outside click.
- `comingSoon(feature)` shows a 2.5s floating toast for unbuilt features (Notes, PYQs).

## 9. Responsive behavior

- Breakpoint: `640px`
- Below 640px: nav links hidden, hamburger shown, multi-button action rows (`.actions`) stack vertically
- 640px+: tools grid becomes 2-column, hamburger hidden, full nav links shown
- Tool pages capped at `max-width:560px` and centered regardless of viewport — calculators are always single-column, narrow, mobile-first even on desktop

## 10. SEO/meta setup (don't drop these when rebuilding)

- Title/description targets keyword set: cgpa calculator, sgpa calculator, bunk predictor, attendance calculator, marks percentage calculator, credit based cgpa, AKTU cgpa calculator
- Open Graph + Twitter Card tags both present, pointing at `aktuverse.vercel.app`
- JSON-LD `WebSite` schema in `<head>`
- `theme-color` meta: `#1C3D2B` (matches `--green`)
- Favicon set: 96×96 png, svg, ico, apple-touch-icon, webmanifest — all under `assets/favicon/`
- Google site verification meta tag present
- GA4 (`G-B19HF9ZG7N`) + Microsoft Clarity (`x7idzga33q`) both loaded in `<head>`

## 11. Known gaps / things explicitly NOT done yet

- Notes and PYQs are stubs (toast only, no real content/pages)
- No localStorage — inputs don't persist across reload (earlier dashboard version of this project *did* have localStorage; current single-file rebuild dropped it)
- No dark mode toggle in this version
- No SGPA calculator (per-subject grade input) — only CGPA from already-known SGPAs

## 12. Process notes for future builds

- Iterate in **layers**: structure first, then styling, then JS behavior — confirm each before moving on
- When asking for changes, prefer giving the exact CSS variable or function name to change rather than a vague visual description
- For new tools, copy the existing "multi-row semester" pattern (section 8) rather than inventing a new structure
- Keep all branding/copy decisions consistent with section 6 verbatim text unless deliberately rewriting copy