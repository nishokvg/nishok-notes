# nishok-notes

Evergreen reference notes & cheatsheets — Linux, Git, AI/ML, and whatever else is worth
keeping. This repo is the **single source of truth** for notes; they are also published to my
site at **https://nishokvg.github.io/notes** automatically.

> Journal/blog posts live in the [`nishokvg.github.io`](https://github.com/nishokvg/nishokvg.github.io)
> repo. This repo is **only** for evergreen notes — timeless references, not dated entries.

---

## How a note becomes a page on the site

```
write here (nishok-notes)                published (nishokvg.github.io/notes)
┌──────────────────────────┐             ┌─────────────────────────────────────┐
│ linux-cmd.md             │   push ──▶  │ this repo is checked out by the      │
│ git-basics.md            │  fires a    │ site's CI, markdown → /notes pages,  │
│ assets/diagram.png       │ rebuild     │ assets → served under /notes/...     │
└──────────────────────────┘             └─────────────────────────────────────┘
```

The flow:
1. You add/edit a `.md` note here and push to `main`.
2. `.github/workflows/notify-site.yml` sends a `repository_dispatch` event to the site repo.
3. The site rebuilds, pulling these notes in, and redeploys to GitHub Pages.

**Write a note → push → it shows up on the site.** No manual copying.

---

## Writing a note

Each note is one Markdown file in the repo root. It **must** start with frontmatter:

```yaml
---
title: "Linux / Bash Command Reference"
category: "Linux"
tags: ["bash", "cli", "linux", "interview-prep"]
excerpt: "A systematic Linux/Bash refresher organized by category."
---

Your note body starts here (regular Markdown — headings, tables, code blocks, etc.).
```

| Field      | Required | Notes                                                                 |
| ---------- | -------- | --------------------------------------------------------------------- |
| `title`    | yes      | Shown as the page heading and in the `/notes` list.                   |
| `category` | yes      | Groups the note on the `/notes` index (e.g. `Linux`, `Git`, `AI/ML`). A new value = a new section appears automatically. |
| `tags`     | optional | Coloured pills on the card and note page.                             |
| `excerpt`  | optional | One-line summary shown under the title.                               |

Conventions:
- **Don't add an `# H1`** at the top — the site renders the title from `title`. Start the body
  with `##` headings.
- The **filename is the URL slug**: `linux-cmd.md` → `/notes/linux-cmd`. Use lowercase,
  hyphenated names.
- Markdown tables, fenced code blocks, blockquotes, and lists all render with the site's styling.

---

## Categories

Categories come from the `category` frontmatter field — there's no fixed list. Reuse an existing
value to add to a section, or introduce a new one to start a new section on the `/notes` page.
Current/suggested categories:

- `Linux`
- `Git`
- `AI/ML`
- `Cloud / Kubernetes`

---

## Images & assets

Put images in an `assets/` folder and reference them with an absolute `/notes/...` path:

```markdown
![Architecture](/notes/assets/architecture.png)
```

At build time the site copies non-markdown files into its `public/notes/` directory, so a file
at `assets/architecture.png` here is served at `/notes/assets/architecture.png` on the site.

---

## Repo layout

```
.
├── README.md              ← this file (not published as a note)
├── linux-cmd.md           ← a note
├── <more>.md              ← more notes
├── assets/                ← images referenced by notes (optional)
└── .github/workflows/
    └── notify-site.yml    ← tells the site to rebuild on push
```

`README.md` is intentionally skipped by the site's sync step, so it never shows up as a note.

---

## Previewing locally

To see notes rendered before pushing, check this repo out **next to** the site repo:

```
project/
  nishok-github-io/   ← site
  nishok-notes/       ← this repo
```

Then in the site repo:

```bash
npm run dev      # auto-syncs notes from ../nishok-notes → http://localhost:3000/notes
```

---

## Auto-rebuild setup

`notify-site.yml` needs a secret named `SITE_DISPATCH_TOKEN` (a GitHub token with write access
to the `nishokvg.github.io` repo) configured in this repo's **Settings → Secrets and variables →
Actions**. Without it, pushes here won't auto-trigger a site rebuild — you can still rebuild the
site manually from its Actions tab.
