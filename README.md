# Farryn1.github.io

Personal academic website of **Fangyuan Shi** — M.S. in Operations Research, Georgia Tech.

🌐 Live at **https://farryn1.github.io/**

Built with [Hugo](https://gohugo.io/) and the
[hugo-coder](https://github.com/luizdepra/hugo-coder) theme (embedded at the repo root).

## Local development

```bash
hugo server            # live preview at http://localhost:1313/
hugo --gc --minify     # production build into ./public
```

Requires the **extended** edition of Hugo (v0.124.0+).

## Structure

```
.
├── hugo.toml                 # site config (identity, menu, social links)
├── content/
│   ├── about.md              # bio, education, research interests, skills
│   ├── research.md           # academic experience
│   └── projects/             # project write-ups
│       ├── wildfire.md
│       ├── large-scale-optimization.md
│       ├── llm-optimization.md
│       └── natural-gas-forecasting.md
├── static/
│   ├── images/avatar.jpg     # profile photo
│   ├── images/projects/      # project figures
│   └── files/                # CV PDF
├── layouts/ assets/ i18n/    # hugo-coder theme (do not edit unless customizing)
└── .github/workflows/hugo.yml  # auto-deploy to GitHub Pages
```

## Editing content

- **Text / bio:** edit `content/about.md` and `content/research.md`.
- **Projects:** each `.md` in `content/projects/` is a project page. The list on `/projects/`
  is ordered by the `date` field (newest first). Set `math = true` in a page's front matter to
  enable LaTeX (KaTeX) rendering.
- **Identity / links:** edit `hugo.toml` (`author`, `info` tagline, `[[params.social]]`,
  `[[menu.main]]`).

## Deployment

Pushing to the `main` branch triggers `.github/workflows/hugo.yml`, which builds the site and
publishes it to GitHub Pages. Enable it once under **Settings → Pages → Build and deployment →
Source: GitHub Actions**.

---

Theme: [hugo-coder](https://github.com/luizdepra/hugo-coder) by Luiz de Prá (MIT License).
