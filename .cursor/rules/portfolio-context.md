# Portfolio Project Context

## Overview
Personal portfolio for **Pierre-Olivier Roux-Savelli**, built with Astro 6 + Tailwind CSS 4 + Sharp for image optimization. Deployed to GitHub Pages at `https://piereau.github.io/portfolio-2026/`.

## Tech Stack
- **Astro 6.1.6** (static site, `astro:assets` for images)
- **Tailwind CSS 4.2** via `@tailwindcss/vite` plugin
- **Sharp 0.34** for image optimization
- **pnpm** as package manager
- GitHub Pages deploy via `withastro/action@v6` on `main` branch

## Important: BASE_URL
The site has `base: '/portfolio-2026'` in `astro.config.mjs`. When referencing static assets from `public/`, always use `import.meta.env.BASE_URL` with a trailing slash fix:
```js
const base = import.meta.env.BASE_URL.replace(/\/?$/, '/');
```

## Fonts (Google Fonts, loaded in Layout.astro)
- **Space Grotesk** (400-700) — headings, body text
- **Bricolage Grotesque** (400-700) — skill chip labels in marquee
- **Montaga** — hero name on index page

## File Structure
```
src/
├── assets/
│   ├── bg-texture.png              # tiling background for body
│   ├── instant-city/               # 6 PNG screenshots
│   ├── jo-companion/               # 7 PNG/JPG screenshots
│   ├── robot-mower/                # 4 PNG screenshots + herbobot-demo.mp4 source
│   ├── etale-ta-connaissance/      # 3 PNG screenshots
│   ├── fidjoo/                     # 2 MP4 videos (copied to public/)
│   ├── habille-moi/                # 1 MP4 video (copied to public/)
│   └── logos/                      # skill logos (SVG + PNG)
├── components/
│   ├── Marquee.astro               # scrolling skill chips banner
│   ├── Projects.astro              # 4 horizontal-scroll project sections
│   └── Welcome.astro               # (unused, leftover from Astro starter)
├── layouts/
│   └── Layout.astro                # base HTML layout
├── pages/
│   └── index.astro                 # home page
└── styles/
    └── global.css                  # Tailwind import + theme config
public/
    ├── bg-texture.png              # copy of tiling bg for production
    ├── herbobot-demo.mp4
    ├── sensor-dashboard-demo.mp4
    ├── fidjoo-attrape.mp4
    ├── fidjoo-volant.mp4
    └── habille-moi-vr-demo.mp4
```

## Key Components

### Layout.astro
- Loads Google Fonts (Space Grotesk, Bricolage Grotesque)
- Tiling `bg-texture.png` background on body
- `BASE_URL` trailing-slash fix for asset paths

### index.astro
- Hero section: name (`font-[Montaga]`) + description + Marquee
- Projects component below

### Marquee.astro
- Infinite horizontal CSS animation (`translate3d(-50%, 0, 0)`)
- 13 skill chips with real logos from `src/assets/logos/`
- Each chip has a colored outline matching the brand
- Duplicated items for seamless loop, second set has `aria-hidden`
- Faded edges via CSS `mask-image`
- Pauses on hover

### Projects.astro
- **7 independent horizontal scroll sections**, each using the Scroll-Linked Animations API (`ViewTimeline`)
- Layout: title/description on left (takes ~70vw), images/videos scroll right
- Each project has its own color scheme defined in the `projects` array.

- **Wavy SVG dividers** between sections: filled wave in the next section's color + stroke line in the previous section's accent color
- **Image handling**: `import.meta.glob` for eager loading, `widths={[900, 1400, 1920]}` for responsive srcsets, `decoding="async"`, lazy loading after first 2 images
- **Video handling**: videos live in `public/` and are referenced via `${base}${filename}.mp4`. They get `object-contain` + `project.img_class` + `project.img_wrapper_class` just like images — keep these consistent when adding new videos.
- **Shadow styles**: most projects use `drop-shadow-[0_8px_40px_rgba(0,0,0,0.35)]` on wrapper; Herbobot uses `.shadow-beautiful` (5-layer box-shadow from Figma plugin)
- `will-change: transform` on `.pin-wrap` for smooth GPU compositing
- Section height: `700vh` (controls scroll speed — higher = slower)
- **Browser support**: ViewTimeline works in Chrome/Edge 115+. No polyfill; sections render static in unsupported browsers.

### Project color palette (source of truth)
Each project in the `projects` array has its colors grouped in 4 logical buckets:

1. **Section** — `bg_class` + `bg_hex` (the `<section>` background, also fills the SVG wave under the section)
2. **Title** — `title_class` (just a Tailwind class for `<h2>`)
3. **Description** — `desc_class` (Tailwind class for `<p>`; not reused anywhere else)
4. **Tag palette** — `tag_class` + 3 hex fields matching its 3 constituent colors:
   - `tag_text_hex` — drives the **active nav dot**
   - `tag_bg_hex` — drives the **inactive nav dots**
   - `tag_stroke_hex` — drives the **stroke of the wave divider above the next section**

The tag palette thus acts as the "accent family" for the whole project: you see those exact 3 colors on the tag pills, and they're echoed in the fixed nav and the wave dividers.

**Naming convention**: fields ending in `_class` hold Tailwind class strings; fields ending in `_hex` hold raw hex values (needed for SVG attrs and inline JS-set styles).

| Field | Type | Purpose |
|---|---|---|
| `bg_class` | class | Tailwind class on the `<section>` |
| `bg_hex` | hex | Hex equivalent — `fill` of the SVG wave under the section |
| `title_class` | class | Tailwind class for `<h2>` |
| `desc_class` | class | Tailwind class for `<p>` description |
| `tag_class` | class | Full Tailwind string for tag pills: `text-X bg-Y/Z border border-W` |
| `tag_text_hex` | hex | Hex of the `text-` shade in `tag_class` — color of the **active nav dot** |
| `tag_bg_hex` | hex | Hex of the `bg-` shade in `tag_class` (full opacity, ignoring `/50` / `/80`) — color of the **inactive nav dots** |
| `tag_stroke_hex` | hex | Hex of the `border-` shade in `tag_class` — color of the **wave stroke above the next section** |
| `img_wrapper_class` | class | Classes on the `<div>` wrapping each image/video (usually a `drop-shadow`) |
| `img_class` | class | Classes on the `<img>`/`<video>` itself (e.g. `shadow-beautiful`) |

#### Current mapping

| Project | `bg_class` / `bg_hex` | `tag_class` (text / bg / border) | `tag_text_hex` (active dot) | `tag_bg_hex` (inactive dots) | `tag_stroke_hex` (wave stroke) |
|---|---|---|---|---|---|
| Instant City | bg-green-950 / #052e16 | emerald-300 / emerald-900/50 / emerald-300 | #6ee7b7 (emerald-300) | #064e3b (emerald-900) | #6ee7b7 (emerald-300) |
| Olympic Companion | bg-blue-950 / #172554 | cyan-300 / cyan-900/50 / cyan-300 | #67e8f9 (cyan-300) | #164e63 (cyan-900) | #67e8f9 (cyan-300) |
| Herbobot | bg-sky-200 / #bae6fd | sky-300 / sky-900 / sky-300 | #7dd3fc (sky-300) | #0c4a6e (sky-900) | #7dd3fc (sky-300) |
| Étale ta connaissance | bg-violet-950 / #2e1065 | violet-200 / violet-900/50 / violet-200 | #ddd6fe (violet-200) | #4c1d95 (violet-900) | #ddd6fe (violet-200) |
| Fidjoo | bg-amber-50 / #fffbeb | amber-900 / amber-200/80 / amber-400 | #78350f (amber-900) | #fde68a (amber-200) | #fbbf24 (amber-400) |
| Dashboard Météo | bg-zinc-100 / #f4f4f5 | zinc-800 / zinc-200 / zinc-400 | #27272a (zinc-800) | #e4e4e7 (zinc-200) | #a1a1aa (zinc-400) |
| Habille-moi VR | bg-fuchsia-50 / #fdf4ff | fuchsia-900 / fuchsia-200/80 / fuchsia-400 | #701a75 (fuchsia-900) | #f5d0fe (fuchsia-200) | #e879f9 (fuchsia-400) |

#### How the tag palette plumbs through the page
- Colors are piped through HTML data attributes:
  - `<section data-dot-bg={project.tag_bg_hex}>` — picked up by the nav observer to color inactive dots
  - `<a data-nav-dot data-accent={project.tag_text_hex}>` — the active dot reads this
  - The SVG divider receives `stroke={projects[idx - 1].tag_stroke_hex}` directly at build time
- An `IntersectionObserver` with `rootMargin: '-50% 0px -50% 0px'` detects which section crosses the viewport centerline, then `setActive(idx, inactiveBg)` applies inline `backgroundColor` on all dots.
- Transitions are smooth thanks to a 0.3s `background-color` transition on `.dot`.

#### Adding a new project — checklist
1. Pick a primary Tailwind color family (e.g. `teal`).
2. Compose `tag_class` first — this is where all the accent logic stems from. It needs exactly 3 Tailwind shades: one for `text-`, one for `bg-` (optionally with `/50` or `/80`), one for `border-`.
3. Set `tag_text_hex`, `tag_bg_hex`, `tag_stroke_hex` to the solid hex of those exact same shades (ignore any `/50` / `/80` — dots and strokes need full opacity).
4. Then fill `bg_class` + `bg_hex`, `title_class`, `desc_class`, `img_wrapper_class`, `img_class`. Rules of thumb:
   - **Dark sections**: `bg_class = bg-<color>-950`, `title_class = text-white`, `desc_class = text-<color>-200/300`. Tag shades typically `text-<color>-200/300`, `bg-<color>-900/50`, `border-<color>-200/300` (text = border).
   - **Light sections**: `bg_class = bg-<color>-50/100/200`, `title_class = text-<color>-950`, `desc_class = text-<color>-800`. Tag shades typically `text-<color>-900`, `bg-<color>-200/80`, `border-<color>-400` (three distinct shades).
5. Look up hex values from the official Tailwind palette: https://tailwindcss.com/docs/customizing-colors
6. Add images via `import.meta.glob('/src/assets/<slug>/*.png', { eager: true })` at the top of the file, or use `video: 'file.mp4'` / `videos: ['a.mp4', 'b.mp4']` with files copied into `public/`.
7. No code changes needed beyond the `projects` array — the nav, the wave dividers, and everything else rebuild automatically.

### Fixed left-side project navigation
- `<nav class="project-nav">` rendered once at the top of `Projects.astro`, `position: fixed`, vertically centered, with a translucent `backdrop-blur` pill background.
- One `<a data-nav-dot>` per project, with `href="#project-{idx}"` + smooth scroll on click.
- Hidden until the user scrolls into the first section (`sections[0]`). An `IntersectionObserver` with `rootMargin: '0px 0px -100% 0px'` toggles `.is-visible` (opacity + pointer-events).
- Hover tooltip shows the project title (hidden on mobile `< 640px`).
- Active dot scales `1.8x` and takes `tag_text_hex` (same as tag pill text); inactive dots take the current project's `tag_bg_hex` (same as tag pill background). See the **Project color palette** section for details.

## Patterns & Conventions
- All project images use **kebab-case** filenames with numeric prefixes (e.g. `01-visualisez.png`)
- Colors are defined per-project as both Tailwind classes AND hex values (hex needed for SVG fills + inline nav dot colors)
- `import.meta.glob` with `{ eager: true }` for build-time image imports
- Tags/chips use `rounded-[6px]` with colored border outlines
- Videos live in `public/`; images live in `src/assets/` and are processed by Astro

## What's NOT done yet
- No footer/contact section
- No "about" section
- No mobile responsiveness testing done
- No favicon customization
