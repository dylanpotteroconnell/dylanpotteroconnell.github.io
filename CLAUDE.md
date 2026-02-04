# Personal Website — dylanoconnell.com

## Overview
Minimalist personal website for Dylan O'Connell, a data scientist. Static site with no frameworks — plain HTML, CSS, and JS only. Hosted on GitHub Pages with a custom domain.

## Owner
- **Name**: Dylan O'Connell
- **GitHub**: dylanpotteroconnell
- **Email**: dylan.potter.oconnell@gmail.com
- **LinkedIn**: https://www.linkedin.com/in/dylan-potter-oconnell/

## Design
- Light, clean, professional aesthetic
- Minimal and low-maintenance
- Mobile-responsive
- No bio/tagline — just name, email, LinkedIn, and content sections

## Structure
```
personal_site/
├── index.html              # Landing page: name, contact, project list
├── styles.css              # Global styles
├── projects/
│   └── oscars-speech-game/ # Self-contained project (user-managed files)
│       ├── index.html
│       └── data.json
├── blog/                   # Future: individual post HTML files
├── CNAME                   # GitHub Pages custom domain config (added when domain is purchased)
└── CLAUDE.md
```

## Key Decisions
- **Repo**: `dylanpotteroconnell/dylanpotteroconnell.github.io` (renamed from `personal_site`)
- **Hosting**: GitHub Pages at https://dylanpotteroconnell.github.io/ (deploy via `git push` to main branch)
- **Domain**: dylanoconnell.com (to be purchased later, likely via Cloudflare Registrar)
- **Contact**: Email + LinkedIn on landing page
- **Sections**: Projects (active), Blog/Writing (planned for future)
- **Projects**: Each project lives in its own subfolder under `projects/`. Currently only oscars-speech-game.
- **Blog/LaTeX**: Blog posts will use KaTeX (via CDN) for equation rendering. No build step needed.

## Rules
- Do NOT generate files inside `projects/oscars-speech-game/` — the user will add those manually.
- Keep everything static and simple. No build tools, no frameworks, no npm.
- Avoid over-engineering. New sections (blog, etc.) should be added only when needed.
- User is not very familiar with git — provide clear git instructions when needed.
