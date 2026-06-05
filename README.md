# 🔨 tagsmith

> Craft polished GitHub releases — changelog, tag, push and GitHub Release in one command.

`tagsmith` is a single, repo-agnostic script you drop on your `PATH` and run from **any
GitHub repository**. It builds a professional `CHANGELOG.md` (commits grouped by type, with
**Contributors** and **New Contributors** resolved to real `@usernames` via the GitHub API),
creates the annotated tag, pushes it, and publishes a **GitHub Release** with the changelog
section as its notes.

## ✨ What you get

- **One command, full release** — changelog → commit → tag → push → GitHub Release.
- **Professional changelog** — commits grouped by type (🚀 Features, 🐛 Bug Fixes, …) with
  scopes, breaking-change markers, and auto-linked `#123` issue/PR references.
- **Real contributor credit** — a ❤️ Contributors and 🌟 New Contributors section built from
  the GitHub API, so commits by the same person under different emails collapse into a single
  `@username`, and first-time contributors are highlighted.
- **Repo-agnostic** — owner / repo / URL are detected from the `origin` remote. No per-project
  config to copy around: one `tagsmith` + one `cliff.toml`.
- **Safe by default** — `--dry-run` previews without touching anything; the real release asks
  for confirmation before it commits, tags, pushes, or publishes.

## 📋 Sample output

```markdown
## [0.4.0](https://github.com/sanbricio/goconcurrencylint/releases/tag/v0.4.0) - 2026-06-05

### 🚀 Features

- Handle parentheses in mutex method calls and add test cases ([#47](https://github.com/sanbricio/goconcurrencylint/issues/47))

### 🐛 Bug Fixes

- *(waitgroup)* Only flag Add(0) for compile-time constants
- *(mutex)* Stop flagging cross-goroutine unlock handoffs ([#46](https://github.com/sanbricio/goconcurrencylint/issues/46))

### 🌟 New Contributors

- @Rafael24595 made their first contribution in [#47](https://github.com/sanbricio/goconcurrencylint/pull/47)

### ❤️ Contributors

- @sanbricio
- @Rafael24595

**Full Changelog**: https://github.com/sanbricio/goconcurrencylint/compare/v0.3.0...v0.4.0
```

## 🔧 Requirements

| Tool | Why | Install |
|------|-----|---------|
| [git-cliff](https://git-cliff.org) | Generates the changelog | `brew install git-cliff` |
| [GitHub CLI (`gh`)](https://cli.github.com), authenticated | Provides the API token and publishes the Release | `brew install gh && gh auth login` |
| Git remote `origin` on GitHub | Owner/repo detection, links, contributors | — |

> The GitHub token is taken automatically from `gh auth token`. You can override it with the
> `GITHUB_TOKEN` (or `GH_TOKEN`) environment variable.

## 📦 Installation

```bash
# 1. Clone wherever you like
git clone <your-fork-or-repo-url> /path/of/your/choice

# 2. Add it to your PATH (zsh)
echo 'export PATH="/path/of/your/choice:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 3. Verify
tagsmith --help
```

(For bash, use `~/.bashrc` instead of `~/.zshrc`.)

## 🚀 Usage

From **any** GitHub repository:

```bash
# Full release: changelog → commit → tag → push → GitHub Release
tagsmith 0.4.0

# Preview the changelog only — no changes, works even with a dirty tree
tagsmith 0.4.0 --dry-run

# Generate the very first CHANGELOG.md from your full history
tagsmith 1.0.0 --init
```

Versions may be passed with or without a leading `v` and support pre-release / build metadata:

```
1.2.3   v1.2.3   1.2.3-rc.1   1.2.3-beta   1.2.3+build.5
```

The changelog always shows the bare version (`1.2.3`); the git tag always carries the `v`
prefix (`v1.2.3`). A version with a pre-release identifier (e.g. `-rc.1`) is published as a
GitHub **pre-release** automatically.

## 🧭 What a release does

```
tagsmith 0.4.0
```

1. ✅ Checks `git-cliff` and `gh` are available
2. 🔍 Detects owner / repo / URL from `origin`, and uses your `gh` token for the GitHub API
3. ✅ Verifies the tag `v0.4.0` doesn't already exist
4. 🔄 Switches to `main`/`master` and pulls (real release only — requires a clean tree)
5. 📝 Generates / prepends `CHANGELOG.md` with `git-cliff`
6. 📋 Shows a preview and offers to open it in your editor
7. 🙋 Asks for confirmation, then `git commit` + `git tag -a v0.4.0`
8. 🚀 `git push` the branch and the tag
9. 🏷️ `gh release create v0.4.0` with the changelog section as release notes

## ⚙️ How it works

- **`tagsmith`** — the script (CLI, preflight, sync, prompts, tag/push, GitHub Release).
- **`cliff.toml`** — the centralized [git-cliff](https://git-cliff.org) config: the changelog
  template, the commit groups, and the skip rules (merges, `chore(release)`, dependency bumps,
  renovate/dependabot, …).

`tagsmith` bakes the detected repo URL into a throwaway copy of `cliff.toml` at run time (the
`__PROJECT_URL__` placeholder), so issue/PR/compare/release links always point at the right
project. When a GitHub token is available it runs **online** (real `@usernames`, contributors,
first-time contributors); without one it falls back to **offline**, attributing each commit to
its git author name instead.

## 🩺 Troubleshooting

| Problem | Solution |
|---------|----------|
| `git-cliff is not installed` | `brew install git-cliff` (or see git-cliff install docs). |
| `GitHub CLI (gh) is not installed` | `brew install gh`. |
| `This release flow targets GitHub …` | Run `gh auth login`, and make sure `origin` points at a GitHub repo. |
| Contributors show as names, not `@usernames` | You're in offline mode — authenticate with `gh auth login` (or set `GITHUB_TOKEN`). |
| `Tag already exists` | You already released that version. Use a higher one. |
| `You have uncommitted changes` | Only the real release needs a clean tree. Commit/stash, or use `--dry-run` to preview. |
| Broken emojis | Make sure your terminal is UTF-8. |

## 🔄 Updating

```bash
cd /path/where/you/cloned/tagsmith && git pull
```
