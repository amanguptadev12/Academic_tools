# AKTU Hub

A zero-bloat, single-file academic toolkit for AKTU (Dr. A.P.J. Abdul Kalam Technical University) students. No backend, no build step, no dependencies — just one HTML file with everything bundled in.

**Live:** [aktuverse.vercel.app](https://aktuverse.vercel.app)

## Tools

- **Attendance Calculator** — how many more classes you need to attend to hit your target %
- **Bunk Predictor** — how many classes you can safely skip without dropping below target
- **CGPA Calculator** — simple average of SGPAs across semesters
- **Credit-Based CGPA Calculator** — weighted CGPA using SGPA × credits per semester
- **Percentage Calculator** — overall marks percentage across semesters

Resources (Notes, PYQs) and quick links (ERP, One View, official site, syllabus) are also on the home page — resources are marked "coming soon."

## Tech

- Pure HTML/CSS/vanilla JS — single file, no frameworks, no build tools
- Page-swap navigation (`showPage()`) with `history.pushState` + hash-based deep linking
- All calculations done client-side, no data leaves the browser
- Google Analytics + Microsoft Clarity for usage insights
- JSON-LD structured data + full Open Graph/Twitter meta for SEO
- Responsive layout: 2-column grid on desktop, stacked on mobile with hamburger nav

## Design

- Green-toned CSS variable system (`--green`, `--green-mid`, `--green-light`, etc.) for easy theme tweaks
- Inter font, soft shadows, rounded cards — minimal/clean aesthetic
- All theme values centralized in `:root` at the top of the `<style>` block

## Running locally

No build step needed — just open the file:

https://aktuverse.vercel.app

## Deployment

Deployed on [Vercel](https://vercel.com) directly from this GitHub repo. Push to `main` → auto-deploys.


