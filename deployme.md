# Deployment — mesabo.github.io

This repository hosts both the jemdoc **source** (on `develop` and `release`) and the **built output** (on `main`, which GitHub Pages serves from `https://mesabo.github.io`).

## Branch model

| Branch | Contents | Purpose |
|---|---|---|
| `develop` | jemdoc source, `assets/`, `Makefile`, etc. | **Working branch.** Daily edits land here. Pushing to `develop` does **NOT** trigger a deploy. |
| `release` | identical layout to `develop` (source) | **Deploy trigger.** Merging `develop` → `release` and pushing fires the GitHub Action, which builds and updates `main`. |
| `main` | rendered HTML (the contents of `build/`) | What GitHub Pages serves at `https://mesabo.github.io`. Maintained by the Action — do not edit by hand. |

## Standard workflow

```bash
# work on develop
git checkout develop
# edit jemdoc / Makefile / assets / etc.
make serve                                # local preview at http://localhost:8000
git add -A && git commit -m "describe change"
git push origin develop

# when ready to publish
git checkout release
git merge develop --ff-only               # or: git rebase develop
git push origin release                   # ← triggers the GitHub Action
```

The Action builds the site on a clean Ubuntu runner and force-pushes the rendered `build/` contents to `main`. The live site updates within ~1 minute after the Action completes.

## Manual deploy (fallback when GitHub Actions is unavailable)

If the Action is broken or you need a local emergency deploy, the old `make deploy` target still works. It uses a persistent clone under `.deploy/` (gitignored) and pushes built HTML directly to `main`, bypassing the Action.

```bash
make clean
make
make deploy                                # local commit + push to main
```

`make deploy` also creates a `rollback-YYYYMMDD-HHMMSS` tag on `main` before changing anything, so a bad deploy can be reverted with:

```bash
make rollback                              # revert the most recent deploy
make rollback-to TAG=rollback-2026...       # revert to a specific tag
make rollback-list                          # show available tags
```

Note: when `make deploy` succeeds it produces the same `main` commit that the Action would have produced from the same source. The two paths are interchangeable.

## What lives where

```
develop  / release  ──────  jemdoc source (this repo's working tree)
                           ├── *.jemdoc, MENU, mysite.conf, mysite.css, Makefile
                           ├── assets/   photos, CVs, papers, reports, awards
                           ├── tools/    vendored jemdoc + gen_gallery.py
                           ├── fr/       French page sources
                           ├── ja/       Japanese page sources
                           └── .github/workflows/deploy.yml

main                ──────  rendered HTML (build/ contents at repo root)
                           served by GitHub Pages at https://mesabo.github.io
```

## Local preview

```bash
make            # build into build/
make serve      # serve build/ at http://localhost:8000
make clean      # remove build/
```

## GitHub Pages settings

The Pages source for this repo is `main` branch, `/` root. The GitHub Action force-pushes the built site there; no Pages configuration change is needed when iterating.

If you ever want to switch GH Pages to use the Action's native `actions/deploy-pages` flow (no main-branch round-trip), edit `.github/workflows/deploy.yml` to use `actions/upload-pages-artifact` + `actions/deploy-pages` and set Pages source to "GitHub Actions" in repo Settings → Pages.

## Common operations

| Task | Command |
|---|---|
| Local preview | `make serve` |
| Build only | `make` |
| Manual deploy (fallback) | `make deploy` |
| Trigger Action deploy | `git push origin release` |
| Roll back last Action deploy | reset `main` to previous commit, then re-merge `develop` to `release` |
| Roll back via `make deploy` flow | `make rollback` |
| List rollback tags | `make rollback-list` |
| Show deploy history | `make log` |
