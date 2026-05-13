# AGENTS.md

## Tech Stack
- Hexo 7.3.0 static blog with Butterfly theme (v5.5.4)
- npm (package manager - pnpm not available in this environment)

## Commands
- `npm run build` or `npx hexo generate` - Generate static site to `public/`
- `npm run deploy` or `npx hexo deploy` - Build and push to GitHub Pages
- `npm run server` or `npx hexo server` - Local dev server (localhost:4000)
- `npm run clean` or `npx hexo clean` - Clear cache and generated files

## Structure
- Posts: `source/_posts/*.md`
- Custom pages: `source/projects/`, `source/about/`
- Theme config: `_config.butterfly.yml` (custom), `_config.yml` (Hexo main)
- Images: `source/img/`

## Theme Configuration
- Theme: Butterfly (replaced Fluid)
- Custom config: `_config.butterfly.yml`
- Search: local_search enabled
- Comments: Disqus (configured)
- Analytics: Google Analytics

## Workflow
1. Create/edit post in `source/_posts/`
2. Run `npx hexo clean && npx hexo generate` to build
3. Run `npx hexo deploy` to publish

## Notes
- Required plugins: hexo-wordcount, hexo-generator-search (already installed)
- Images copied from Fluid theme to `source/img/` for compatibility
- Life category posts go in `source/_posts/` with `categories: [life]`