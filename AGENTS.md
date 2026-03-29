# AGENTS.md

## Site

Hugo static site using the [Blowfish](https://github.com/nunocoracao/blowfish) theme (v2), managed as a Go module. Custom domain: `nikimanoledaki.com`.

## Content

All pages live under `content/`. Each section has an `_index.md`:

- `content/_index.md` — homepage bio
- `content/about/_index.md` — about page bio
- `content/talks/_index.md` — conference talks
- `content/community/_index.md` — meetups and community work
- `content/media/_index.md` — media appearances
- `content/podcasts/_index.md` — podcast episodes
- `content/posts/` — blog posts (each in its own subdirectory)

The homepage and about page bios should stay in sync.

## Configuration

Hugo config lives in `config/_default/`:

- `hugo.toml` — base config, module imports
- `params.toml` — theme parameters (color scheme, layout)
- `languages.en.toml` — author info, social links
- `menus.en.toml` — navigation menu

Layout overrides are in `layouts/partials/` (custom hero, Google Fonts).

## Publishing

1. Edit content in `content/` or config in `config/_default/`
2. Commit and push to `master`
3. GitHub Actions (`.github/workflows/hugo.yml`) builds with Hugo 0.142.0 and deploys to GitHub Pages

No local build step is required. The `public/` directory is gitignored; CI generates it fresh on each deploy.

## Local preview

```
hugo server -D
```

## Commits

Follow the format in the root `CLAUDE.md`: `type(scope): description`. No AI attribution lines.
