# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an academic research project page template built with Astro 5, MDX, React, and Tailwind CSS v4. It renders a single-page research paper website from `src/paper.mdx` with features like LaTeX math rendering, PDF-to-image conversion, dark/light themes, and interactive components (comparison sliders, carousels, 3D model viewers, etc.).

## Development Commands

```bash
# Setup
npm install

# Development server (opens at http://localhost:4321)
npm run dev

# Type checking
npm run check

# Build (runs typecheck then builds to dist/)
npm run build

# Preview built site
npm run preview

# Linting
npm run lint          # Check ESLint
npm run lint:fix      # Auto-fix ESLint issues
npm run format        # Check Prettier
npm run format:fix    # Apply Prettier formatting
npm run all:fix       # Run check + lint:fix + format:fix
```

## Architecture

### Content Structure
- **Main content**: `src/paper.mdx` - MDX file with YAML frontmatter containing all page content
- **Frontmatter keys**: `title`, `authors`, `conference`, `notes`, `links`, `description`, `favicon`, `thumbnail`, `theme`
- **Layout**: `src/pages/index.astro` - Imports `{ Content, frontmatter }` from `src/paper.mdx` and provides HTML structure, OpenGraph tags, and theme handling

### Key Components
- `Picture.astro` - Image display component that supports both imported images (Astro's `ImageMetadata`) and PDF files (auto-converts PDFs to PNG via `src/lib/render-pdf.ts`)
- `Figure.astro` - Wraps visuals with captions using named slots: `figure` and `caption`
- `Video.astro`, `YouTubeVideo.astro`, `ModelViewer.astro` - Media display components
- `Carousel.astro`/`CarouselSlide.astro` - Image carousel with swipe/keyboard navigation
- `Comparison.tsx` - React before/after slider (requires `client:*` hydration directive)
- `TwoColumns.astro`, `Wide.astro`, `HighlightedSection.astro`, `SmallCaps.astro` - Layout utilities
- `starwind/tabs` - Tabbed content panels

### Tech Stack
- **Astro 5** - Static site generator with islands architecture
- **MDX** - Markdown with JSX components
- **React 19** - For interactive client-side components
- **Tailwind CSS v4** - Styling via `@tailwindcss/vite`
- **TypeScript** - Strict mode with `@/*` → `./src/*` path alias
- **KaTeX** - LaTeX math rendering via `remark-math` + `rehype-katex` (use `$...$` inline, `$$...$$` block)

### Theme System
- Set via `theme` in frontmatter: `device` (default), `light`, or `dark`
- Layout writes `data-theme` attribute on `<html>`
- Tailwind's custom `dark` variant reads from `data-theme`
- Use `dark:*` utilities for dark-mode-specific styles
- `<Picture>` component supports `invertInDarkMode` prop

### Asset Handling
- **Public assets**: Files in `public/` served at base URL
- **Imported images**: Use `import image from "./assets/file.png"` for optimization
- **PDF files**: Pass relative path string to `<Picture src="../assets/figure.pdf" />` - auto-converts page 1 to PNG at 4× scale during build/dev
- **URL prefixing**: Use `import.meta.env.BASE_URL` for absolute URLs in layout/components

### Path Resolution
- PDF paths in `<Picture>` are resolved relative to `./src/pages/`
- During dev, converted PDFs written to `dist/_astro/*.png`
- During production build, written to `dist/_astro/*.png`

## Component Usage Patterns

### Adding content
1. Edit `src/paper.mdx` for all page content
2. Import components at the top of the file
3. For React components, add hydration directive where used: `<Comparison client:idle>...</Comparison>`

### Using components
```jsx
// Figure with caption
<Figure>
  <Picture slot="figure" src={imageImport} alt="Description" />
  <Fragment slot="caption">Caption text</Fragment>
</Figure>

// Wide breakout visual
<Wide>
  <Picture src={imageImport} alt="Description" />
</Wide>

// Carousel
<Carousel>
  <CarouselSlide>Content 1</CarouselSlide>
  <CarouselSlide>Content 2</CarouselSlide>
</Carousel>

// PDF to image
<Picture src="../assets/figure.pdf" alt="Auto-converted PDF" />
```

### Adding new components
1. Create under `src/components/` (`.astro` for static, `.tsx` for React)
2. Import in `src/paper.mdx`
3. For React components, add `client:*` directive at usage site (e.g., `client:idle`, `client:load`, `client:visible`)

## Deployment
- **GitHub Pages** (default): Push to `main` branch triggers `.github/workflows/astro.yml`
- The workflow passes `--site`/`--base` for correct asset paths
- Layout already handles `import.meta.env.BASE_URL` for favicon/OpenGraph tags

## Linting and Formatting
- **ESLint**: Configured for JS/TS/TSX, Astro, JSON, Markdown, and CSS (Tailwind v4 syntax)
- **Prettier**: With `prettier-plugin-astro` and `prettier-plugin-tailwindcss`
- Use `npm run all:fix` to run typecheck, lint fixes, and formatting together

## Icons
- Uses `astro-icon` with Iconify sets
- Available sets: `@iconify-json/academicons`, `@iconify-json/ri`
- Example usage in frontmatter: `icon: academicons:arxiv`

## Code Blocks
- Styled via `astro-expressive-code`
- Configuration in `astro.config.ts` - don't manually theme code blocks

## Project Status
- Paper accepted at IROS 2026 (IEEE/RSJ International Conference on Intelligent Robots and Systems)
- Authors: TaeYoung Kim, Gilhwan Kang, Tae Ihn Kim, Seungwon Song, Hun Keon Ko (Hyundai Motor Company Robotics Lab)
- Deployed at: https://taeyoung96.github.io/mtd_map/ (requires repo to be public)

## GitHub Pages
- Repo must be **public** for GitHub Pages to work on free plan
- After making public: Settings → Pages → Source → GitHub Actions
- Remote uses SSH: `git@github.com:Taeyoung96/mtd_map.git`

## SVG Figure Extraction
- Paper figures stored as SVG in `/home/taeyoung/Downloads/IROS2026_clean/Figure/`
- `figure_3.svg` contains 8 panels (2×4 grid) with base64-embedded JPEGs
- Extract panels with Python: parse defs images, map via group transforms `translate(-1356,-229)`
- Panel mapping: panel_2=Ground Truth, panel_3=HMM-MOS, panel_0=ELite, panel_4=MTD-Map(Ours)
- Extracted assets saved to `src/assets/dor_*.jpg`

## Table Styling in MDX
- Wide markdown tables: use HTML `<table>` with `colspan` for grouped headers
- Center table on page: `width:max-content; margin:0 auto`
- `<code>` tags in table headers render with gray background highlight — avoid, use plain text
- `conference` field rendered in `src/pages/index.astro` line 76; use `text-xl font-semibold` class
