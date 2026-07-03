# CLAUDE.md — Robert Book Design Portfolio Site

This file documents the `robert-book-design` repository for future Claude Code sessions. It covers the tech stack, file layout, visual design system, interactive behavior, and content/business context so that edits stay consistent with the existing site.

## 1. What this project is

A **static, framework-free, no-build-step portfolio website** for Robert Book, a freelance graphic designer / illustrator / web designer based in Lynchburg, VA. There is no `package.json`, no bundler, no `.gitignore`, no README — just hand-authored HTML, CSS, and vanilla JS pushed directly to a **GitHub Pages** deployment with a custom domain (`CNAME` → `robertbookdesign.com`).

Despite the repo/domain name containing "book," this is **not** a book-design or publishing business — "Book" is the designer's surname. The brand is literally his name-as-logo.

There is no templating system. All 8 HTML pages hand-duplicate the same `<head>`, navigation, mobile menu, and footer markup. When changing nav/footer/header structure, **you must edit every page individually** — there is no shared partial/include to update once.

## 2. File structure

```
robert-book-design/
├── CNAME                                  → "robertbookdesign.com" (GitHub Pages custom domain)
├── index.html                             → Homepage
├── about.html
├── contact.html
├── projects.html                          → Portfolio index (all case studies)
├── c&l-motors-restoration.html            → Project case study
├── leaf&ladle-restaurant-posters.html     → Project case study
├── mellowstruck-logo-design.html          → Project case study
├── mellowstruck-portfolio-website.html    → Project case study
├── sakura&sea-landing-page.html           → Project case study
├── ward-media-logo.html                   → Project case study
├── css/
│   └── style.css                          → single stylesheet, ~4400 lines, all site CSS
├── fonts/
│   ├── D-DIN.otf                          → body font
│   └── D-DIN-Bold.otf                     → heading/link font
├── images/                                → 30 .webp assets + favicon.ico
└── Java/                                  → JS folder (name is a joke/typo for "JavaScript", not the language Java)
    ├── breadcrumb.js                      → scroll-driven breadcrumb trail
    └── hamburger-menu.js                  → mobile nav toggle
```

No icon font or SVG icon set is used anywhere — even small UI icons (address/email/LinkedIn glyphs on the contact page) are raster `.webp` images.

## 3. Visual design system

### Aesthetic
High-contrast **black-and-white brutalist/technical** design: thick black borders, large rounded corners, giant **outlined** (stroke-only, transparent-fill) display typography, and a single accent color reserved exclusively for hover states. The look is graphic-designer-portfolio-as-billboard — big confident type, minimal color, generous whitespace, sharp geometric structure.

### Color palette
No CSS custom properties/variables are used anywhere in the stylesheet — every color is a hard-coded hex/rgba literal, repeated inline. If you introduce new components, follow this same literal-value convention (or, better, consider introducing CSS variables at the top of `style.css` since none currently exist).

| Color | Hex / value | Usage |
|---|---|---|
| White | `#FFFFFF` | Primary background |
| Black | `#000000` | Primary text/border/ink color, default stroke |
| **Red (signature accent)** | `#DD1212` | **Hover-only.** Border color and text-stroke color swap to this on `:hover`. This is the single interactive accent for the entire site. |
| Grey | `#999999` | Form input placeholder text |
| Shadow | `rgba(0,0,0,0.15)` / `rgba(0,0,0,0.25)` | Drop shadows on images/cards |

### Typography
Two custom `@font-face` weights of a single family, D-Din:
```css
@font-face { font-family: D-Din;      src: url(../fonts/D-DIN.otf); }
@font-face { font-family: D-Din-Bold; src: url(../fonts/D-DIN-Bold.otf); }
```
- `h1`–`h4` and all `a` elements → **D-Din-Bold**
- `p` (body copy) → **D-Din** (regular)
- Fallback: `sans-serif`

**Outline-type motif**: headings frequently use `color: transparent` + `-webkit-text-stroke` / `-moz-text-stroke: 1px #000000` instead of a solid fill color — giant hollow letterforms are a recurring signature look, most visible in the homepage hero (`ROBERT / BOOK / DESIGN`) and every section's title treatment.

**Section title treatment** (repeated identically for About, Contact, and every project case-study block): a large horizontal `<h1>` paired with a smaller **vertically rotated** `<h1>`/`<h2>` (`writing-mode: vertical-rl; transform: rotate(180deg)`), separated by a 2px black "ruler" divider line. This vertical-rotated-label + horizontal-title + divider combo is the site's core recurring layout module — reuse it for any new section that needs the same visual family.

### Signature interaction: the hover-to-red motif
Across cards, links, and borders, the default state is pure black (`#000000`) and on `:hover` the stroke/border swaps to red (`#DD1212`). This single, consistent hover rule is effectively the site's whole "interactivity language" — there are no other hover treatments (no color fills, no underlines-on-hover, no shadows-on-hover as the primary signal).

### Imagery style
Project cards use a zoom-on-hover effect:
```css
transform: scale(1.05) /* or 1.075 */;
transition: transform 0.9s cubic-bezier(0.25, 0.8, 0.25, 1);
```
Photography is real photos of the designer, his workspace, and client project mockups — not stock/illustration-heavy. Each client case study follows a strict image-naming convention: `{ClientName}_Objective.webp`, `{ClientName}_Ideation.webp`, `{ClientName}_Solution.webp`, `{ClientName}_Square.webp` (thumbnail).

### Animation
One custom keyframe animates the homepage hero's auto-scrolling testimonial carousel:
```css
#hero-review-track { animation: review-scroll 15s linear infinite; }
#hero-review-track:hover { animation-play-state: paused; }
@keyframes review-scroll {
  from { transform: translateY(0); }
  to   { transform: translateY(-114rem); }
}
```
The track contains 3 unique testimonial cards duplicated once (6 total DOM nodes) to fake a seamless infinite loop. Below the `1300px` breakpoint the scroll direction flips from vertical (`translateY`) to horizontal (`translateX`), with distance/duration retuned per breakpoint (18s, distances from `-96rem` down to `-63rem`). If you add/remove a testimonial, you must update the duplicate set **and** recalculate the `translateY`/`translateX` distances at every breakpoint or the loop will visibly jump.

Global smooth scroll is enabled (`html { scroll-behavior: smooth; }`), and every anchorable section has `scroll-margin-top` (150–250px) so the sticky header/breadcrumb bar never covers content when jumping to an in-page anchor.

### Responsive breakpoints
Desktop-first, extremely granular — roughly 30 `max-width` media queries stepping from `1800px` down to `400px` (e.g. 1650, 1500, 1450, 1425, 1350, 1300, 1280, 1250, 1220, 1200, 1150, 1120, 1100, 1075, 1050, 1030, 1000, 950, 850, 810, 800, 700, 675, 650, 620, 610–575, 550, 515, 500, 450, 420, 400). At **`1050px`**, the horizontal `#nav-links` menu is hidden and replaced by the `#hamburger` toggle + `#mobile-menu` dropdown. `style.css` mirrors its section order twice — once for base desktop styles, once again in a trailing "RESPONSIVENESS" block — so any new component needs both a base rule and a matching responsive override added in the correct section of each half.

## 4. JavaScript behavior (`Java/` folder — vanilla JS, zero dependencies/libraries)

### `hamburger-menu.js` (6 lines)
Toggles the mobile nav open/closed:
```js
const hamburger = document.getElementById("hamburger");
const mobileMenu = document.getElementById("mobile-menu");
hamburger.addEventListener("click", () => { mobileMenu.classList.toggle("open"); });
```

### `breadcrumb.js` (55 lines)
A fully custom, dependency-free scroll-driven breadcrumb trail. Every major section in the HTML carries `data-label` (its display name) and `data-parent` (its ancestor section's id) attributes. Key functions:
- **`getActiveZone()`** — scans all `data-label` elements bottom-up and returns the last one whose top has scrolled past the viewport's vertical middle; that becomes the "you are here" section.
- **`buildBreadcrumb(zone)`** — walks the `data-parent` chain from the active zone up to the root, then renders each ancestor's `data-label` as a `<a href="#id">LABEL</a>`, joined by `•` bullets, into `#breadcrumb-nav`.
- **`onScroll()`** — recomputes the active zone on every scroll event, only rebuilding the breadcrumb DOM when it actually changes (`lastActiveZone` perf guard).
- Wired via `window.addEventListener("scroll", onScroll)` and an initial call on `DOMContentLoaded`.

**If you add a new section to any page, you must add matching `data-label`/`data-parent` attributes** or it will be invisible to the breadcrumb system.

There is no other custom JS: no form-validation script, no modal/lightbox/slider/filtering logic. The testimonial carousel is CSS-only (see Animation above). The contact form submits directly to a third-party service (Formspree) with no client-side JS involved.

## 5. Page-by-page content map

All pages share identical `#navigation` (logo + `ROBERT BOOK DESIGN` header title + HOME/ABOUT/PROJECTS/CONTACT nav links + hamburger) and footer (`©2025 ROBERT BOOK DESIGN • ALL RIGHTS RESERVED` / "POWERED BY ROBERT BOOK DESIGN" linking to the live domain).

- **`index.html`** — Hero (`ROBERT / BOOK / DESIGN` stacked outline-type headline, "CONTACT ME" + "SEE PROJECTS ▼" CTAs) with an auto-scrolling testimonial carousel beside it → About teaser (portrait + vertical-rotated title + bio) → Featured Works (4 project cards: Mellowstruck Logo, Leaf&Ladle Posters, Mellowstruck Portfolio Website, Sakura&Sea Landing Page) → Contact teaser ("LETS CREATE").
- **`about.html`** — Bio block (same copy as homepage About) + "MY PASSION" block (reversed image/text layout) about personal growth/projects, then a repeated "LETS CREATE" contact teaser.
- **`contact.html`** — Left sidebar: LOCATION (Google Maps link, Lynchburg VA), EMAIL (`robertbookdesign@gmail.com` and `robert@booksoffaith.org`), LINKEDIN (`linkedin.com/in/robert-book-design/`). Right side: a real HTML form posting to **Formspree** (`action="https://formspree.io/f/mwvnbpyp"`, `method="POST"`, `enctype="multipart/form-data"`) with required email/subject/message fields (a file-upload field exists in markup but is commented out/disabled).
- **`projects.html`** — Two grouped sections: "GRAPHIC DESIGN PROJECTS" (Mellowstruck Logo, Leaf&Ladle Posters, C&L Motors Restoration, Ward Media Logo) and "UI/UX DESIGN PROJECTS" (Mellowstruck Portfolio Website, Sakura&Sea Landing Page).
- **Case study pages** (6, one per client) all follow the same numbered 01/02/03 Objective → Ideation → Solution template, each block pairing a large image with vertically-rotated title + two paragraphs of copy:
  - `mellowstruck-logo-design.html` — geometric "M+S" mark for EDM/lofi music producer MellowStruck.
  - `c&l-motors-restoration.html` — rebuilt a pixelated legacy dealership logo into a scalable vector suite (light/dark variants).
  - `leaf&ladle-restaurant-posters.html` — restaurant poster series using **Norwester** (structured) and **Knewave** (sketchy accent) typefaces.
  - `ward-media-logo.html` — clapperboard-mark logo for freelance film producer Hunter Ward; dark blue/red/white palette.
  - `mellowstruck-portfolio-website.html` — 4-page responsive dark-themed site, live at `mellowstruck.com` (has a "VIEW LIVE SITE" link).
  - `sakura&sea-landing-page.html` — self-initiated speculative landing page for a sushi restaurant; dark background, gold serif headline + sans-serif body.

## 6. Business/content context

- **Who**: Robert Book, freelance graphic designer / illustrator / web designer, Lynchburg, VA.
- **Services** (per meta description/bio copy): web design, layout design, illustration, print design, branding, motion graphics.
- **Contact**: `robertbookdesign@gmail.com` (primary) and `robert@booksoffaith.org` (secondary — likely a nonprofit/ministry affiliation, not explained on-site). LinkedIn: `linkedin.com/in/robert-book-design/`.
- **Clients featured**: MellowStruck (music producer — logo + full site, both live), C&L Motors (dealership — logo restoration; testimonial from Cibeles Salomon), Ward Media (Hunter Ward, film producer — logo), Leaf&Ladle (restaurant — poster series), Sakura&Sea (sushi restaurant — speculative/self-initiated project).

## 7. Known quirks worth knowing before editing

- **`Java/` folder name** is just a colloquial nickname for JavaScript, not related to the Java language.
- **Typos in source**: homepage hero eyebrow reads "Graphic Designer • Illistrator • Web Developer" (should be "Illustrator"); `mellowstruck-logo-design.html`'s `<h1>` reads "MELOWSTRUCK LOGO DESIGN" (missing an "L") even though the `<title>` tag spells it correctly.
- **`projects.html` id/data-label bugs**: several project cards for C&L Motors and Ward Media incorrectly reuse the id `mellowstruck-logo-design-project` and `data-label="MELLOWSTRUCK LOGO"` left over from copy-pasting — likely worth fixing if touching that page, but be aware it's pre-existing, not something you broke.
- **No shared partials**: nav/footer/head markup is duplicated across all 8 HTML files by hand. Any global structural change (new nav link, footer text change, meta tag) must be repeated in every file.
- **No CSS variables**: all colors are hard-coded literals repeated throughout `style.css`. Match the existing literal style unless the user asks you to refactor toward variables.
- **No build step**: changes to HTML/CSS/JS are live as soon as pushed to the GitHub Pages branch — there is no compile/minify/bundle step to run.
