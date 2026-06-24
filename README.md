# articwnd.github.io

Personal portfolio site for **Richard Xiong**, built with Astro 6 and deployed via GitHub Pages. Showcases work as a student, developer, and researcher at the intersection of financial technology and data.

**Live site:** https://articwnd.github.io

---

## Overview

A fully static portfolio built with Astro 6's islands architecture. Content is authored in MDX/Markdown via Astro's Content Collections API and deployed automatically on every push to `main` via GitHub Actions.

**Pages:**

- **Home** — Hero section with profile photo, bio, and social links
- **About** — Personal background, interests, and fast facts
- **Experience** — Reverse-chronological work and research history
- **Projects** — Data-driven project listing with individual detail pages
- **Research** — Academic writing and research notes (combined blog/research)
- **Contact** — Contact information and outreach

---

## Tech Stack

| Layer | Tool |
|---|---|
| Framework | Astro 6 |
| Deployment | GitHub Pages via GitHub Actions |
| Content | Astro Content Collections (Markdown) |
| Fonts | Space Grotesk, JetBrains Mono (Google Fonts) |
| Styling | Scoped CSS with global CSS variables |
| Language | TypeScript |

---

## File Structure

```
articwnd.github.io/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions deploy pipeline
├── public/
│   ├── favicon.svg
│   └── resume.pdf              # Served at /resume.pdf
├── src/
│   ├── assets/
│   │   └── profile.jpg         # Processed by Astro image pipeline
│   ├── components/
│   │   └── Nav.astro           # Global fixed navigation bar
│   ├── content/
│   │   └── projects/           # One .md file per project
│   │       └── [project].md
│   ├── layouts/
│   │   └── Layout.astro        # Shared <html>, <head>, global CSS, fonts
│   ├── pages/
│   │   ├── index.astro         # Home page
│   │   ├── about.astro         # About page
│   │   ├── experience.astro    # Experience page
│   │   ├── research.astro      # Research/blog page
│   │   ├── contact.astro       # Contact page
│   │   ├── projects.astro      # Projects listing
│   │   └── projects/
│   │       └── [id].astro      # Dynamic individual project pages
│   └── content.config.ts       # Astro Content Collections schema
├── astro.config.mjs            # Astro config (site URL, output mode)
├── package.json
├── tsconfig.json
└── README.md
```

---

## Adding a New Project

Create a new `.md` file in `src/content/projects/`. The filename becomes the URL slug (e.g., `heatmap-visualization.md` resolves to `/projects/heatmap-visualization`).

**Required frontmatter:**

```markdown
---
title: "Project Title"
description: "One sentence on what it does and why it matters."
tech: ["Python", "Pandas", "scikit-learn"]
github_link: "https://github.com/articwnd/repo-name"
demo_link: "https://your-demo-url.com"
year: 2026
featured: true
---

## Problem
What problem were you solving?

## Approach
Key technical decisions and why.

## Tradeoffs
What didn't work or what you'd do differently.

## Outcome
Results, metrics, or what you learned.
```

`github_link` and `demo_link` are optional. All other fields are required and schema-validated at build time.

---

## Local Development

```bash
npm install
npm run dev       # starts dev server at localhost:4321
npm run build     # production build to dist/
npm run preview   # preview production build locally
```

**Note:** If `npm run dev` shows content collection errors on Windows, the dev server occasionally reports empty collections due to a known Astro/Windows file-watching bug. Run `npm run build && npm run preview` as a reliable alternative.

---

## Deploy

Deployment is fully automated. Every push to `main` triggers `.github/workflows/deploy.yml`, which builds the site and deploys the `dist/` output to GitHub Pages.

**Manual trigger:** navigate to the Actions tab in the repo and run the workflow manually via `workflow_dispatch`.

GitHub Pages source must be set to **GitHub Actions** (not "Deploy from branch") in repo Settings for the workflow to take effect.

---

## Design Tokens

Defined once in `Layout.astro` as global CSS variables, available on every page.

```css
--bg:      #030B18   /* page background */
--border:  #1B2D4A   /* dividers, card borders */
--primary: #E8EDF5   /* primary text */
--muted:   #6B8099   /* secondary text */
--accent:  #38BDF8   /* highlights, links, tags */
```

---

## Content Notes

- Resume is served directly from `public/resume.pdf`. To update, overwrite the file and push. No link changes needed.
- Profile photo lives in `src/assets/` and is processed by Astro's image optimization pipeline at build time.
- Navigation order is controlled by the `links` array in `src/components/Nav.astro`.
- Experience entries are data-driven via a `const experiences = [...]` array in `src/pages/experience.astro`. Add new entries to the top of the array to maintain reverse-chronological order.
