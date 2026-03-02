# lab.wcloud.sh

Hugo static site documenting a homelab Kubernetes cluster.

## Stack

- Hugo with Blowfish v2 theme (Hugo module via `go.mod`)
- Deployed to GitHub Pages via GitHub Actions
- Custom OLED color scheme (`assets/css/schemes/oled.css`)

## Local Development

```
hugo serve
```

## Build

```
hugo --minify
```

## Structure

- `config/_default/` - Hugo config (TOML): `hugo.toml`, `params.toml`, `menus.en.toml`, etc.
- `content/` - Markdown content with TOML frontmatter
  - `infrastructure/` - Cluster architecture docs
  - `services/` - Service documentation
  - `posts/` - Blog/journal entries (use page bundles: `posts/<slug>/index.md`)
- `assets/css/schemes/` - Custom color schemes
- `static/` - Static files (includes CNAME)

## Conventions

- Mermaid diagrams use Blowfish shortcodes: `{{</* mermaid */>}}` not fenced code blocks
- Dark mode only (OLED scheme, no appearance switcher)
- Content frontmatter is TOML (`+++`)
- New blog posts go in `content/posts/<slug>/index.md` (page bundle)
