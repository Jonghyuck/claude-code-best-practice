---
paths:
  - "**/*.md"
---

# Korean Translation Sync (한글 번역본 동기화)

This repository maintains Korean translations alongside the English source
docs. Translated files use the `.kor.md` suffix next to their English
originals (e.g. `README.md` → `README.kor.md`, `CLAUDE.md` → `CLAUDE.kor.md`,
`best-practice/claude-subagents.md` → `best-practice/claude-subagents.kor.md`).

## Rules

- **English is the source of truth.** `.kor.md` files are translations that
  follow their English original, never the reverse.
- **When you edit an English `X.md`, update the matching `X.kor.md`** in the
  same change so the translation does not drift. If `X.kor.md` does not yet
  exist, it is fine to leave it for a later translation pass — do not block
  the English edit on it.
- **Do not translate a `.kor.md` back into `X.md`.** If a `.kor.md` is edited
  directly, treat that edit as pending translation review, not as new source.
- **Preserve structure and non-prose markup.** Keep headings, tables, badge
  image tags (`!/tags/...`), links, anchors, and code blocks identical to the
  English original. Translate prose, table cell text, and descriptions only.
- **Keep links pointing at English targets** unless a `.kor.md` counterpart of
  the linked file also exists.
- **Per-file commits.** Follow the repository's git rule: commit each file
  (English or Korean) separately with a descriptive message.

## Naming convention

`<name>.kor.md` — chosen over `<name>(kor).md` to avoid shell/URL escaping of
parentheses. If the convention changes, update this rule file first.
