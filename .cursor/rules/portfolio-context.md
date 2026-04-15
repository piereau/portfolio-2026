# Portfolio Project Context

## Overview
Personal portfolio for **Pierre-Olivier Roux-Savelli**, built with Astro 6 + Tailwind CSS 4 + Sharp for image optimization. Deployed to GitHub Pages at `https://piereau.github.io/portfolio-2026/`.

Inspiration: https://salim.altplus.dev/en

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
│   ├── robot-mower/                # 4 PNG screenshots
│   ├── etale-ta-connaissance/      # 3 PNG screenshots
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
    └── bg-texture.png              # copy of tiling bg for production
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
- **4 independent horizontal scroll sections**, each using the Scroll-Linked Animations API (`ViewTimeline`)
- Layout: title/description on left (takes ~70vw), images scroll right
- Each project has its own color scheme defined in the `projects` array:

| Project | bg class | bg_fill (hex) | wave_stroke | Text colors |
|---|---|---|---|---|
| Instant City | bg-green-950 | #052e16 | #34d399 | white / emerald |
| Winter Olympics Companion | bg-blue-950 | #172554 | #22d3ee | white / cyan |
| Herbobot | bg-sky-200 | #bae6fd | #0284c7 | sky-950 / sky-800 |
| Étale ta connaissance | bg-violet-950 | #2e1065 | #a78bfa | white / violet |

- **Wavy SVG dividers** between sections: filled wave in the next section's color + stroke line in the previous section's accent color
- **Image handling**: `import.meta.glob` for eager loading, `widths={[900, 1400, 1920]}` for responsive srcsets, `decoding="async"`, lazy loading after first 2 images
- **Shadow styles**: most projects use `drop-shadow-[0_8px_40px_rgba(0,0,0,0.35)]` on wrapper; Herbobot uses `.shadow-beautiful` (5-layer box-shadow from Figma plugin)
- `will-change: transform` on `.pin-wrap` for smooth GPU compositing
- Section height: `700vh` (controls scroll speed — higher = slower)
- **Browser support**: ViewTimeline works in Chrome/Edge 115+. No polyfill; sections render static in unsupported browsers.

## Patterns & Conventions
- All project images use **kebab-case** filenames with numeric prefixes (e.g. `01-visualisez.png`)
- Colors are defined per-project as both Tailwind classes AND hex values (hex needed for SVG fills)
- `import.meta.glob` with `{ eager: true }` for build-time image imports
- Tags/chips use `rounded-[6px]` with colored border outlines

## What's NOT done yet
- Hero description is still lorem ipsum
- No navigation/header
- No footer/contact section
- No "about" section
- Welcome.astro is unused (leftover from starter, can be deleted)
- `background.svg` and `astro.svg` in assets are unused leftovers
- Montaga font is referenced in index.astro classes but may not be in the Google Fonts link yet
- No mobile responsiveness testing done
- No favicon customization
