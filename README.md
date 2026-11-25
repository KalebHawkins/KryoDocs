# KryoDocs

Source for [kalebhawkins.github.io/KryoDocs](https://kalebhawkins.github.io/KryoDocs/): write-ups, notes, and a log of certifications, Hack The Box progress, books, and projects.

Built with [Hugo](https://gohugo.io/) and a custom theme in `layouts/`. Every push to `main` builds the site and deploys it to GitHub Pages through `.github/workflows/deploy.yml`.

## Run locally

Requires Hugo extended 0.170 or later.

```
hugo server
```

The site is served at http://localhost:1313/KryoDocs/ and reloads on save.

## Add content

| To add | Do this |
|---|---|
| A write-up | `hugo new writeups/<name>.md`, write it, set `retired: true` and `draft: false` |
| A note | `hugo new notes/<topic>/<name>.md` |
| A timeline entry | Append to `data/timeline.yaml` |
| A cert, book, or role | Edit `data/certs.yaml`, `data/books.yaml`, or `data/career.yaml` |
| New stats | Edit `data/stats.yaml` (kryostats will generate it later) |

Hack The Box only allows public write-ups for retired content. A write-up without `retired: true` fails the build.

## Layout

| Path | Contents |
|---|---|
| `content/` | Markdown pages: write-ups, notes, projects, section intros |
| `data/` | YAML the home page, journey, and bookshelf are built from |
| `layouts/` | Templates (Go `html/template`) |
| `assets/css/main.css` | All styles |
| `archetypes/` | Front matter templates for `hugo new` |
