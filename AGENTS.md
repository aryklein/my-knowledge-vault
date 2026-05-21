# AGENTS.md

## Repository Shape

- This is an Obsidian Markdown knowledge vault, not an application repo; there is no package manifest, CI workflow, or build/test/lint command in the repo.
- Primary content lives under `docs/`, grouped by topic (`applications`, `databases`, `kubernetes`, `linux`, etc.). Diagrams live in `Excalidraw/` as `.excalidraw.md` files.
- Navigation files are root-level Obsidian notes: `INDEX.md`, `TAGS.md`, `TEMPLATES.md`, and `useful_links.md`.
- `.obsidian/` is ignored by `.gitignore` and should be treated as local editor state unless the user explicitly asks to change vault settings.

## Writing Notes

- Preserve Obsidian wiki links such as `[[INDEX|Back to Main Index]]` and `[[PostgreSQL/CloudNativePG|CloudNativePG]]`; do not convert them to standard Markdown links unless linking outside the vault.
- Existing notes commonly use YAML frontmatter with `tags`; both list form and comma-separated scalar form are present, so match the nearby file style instead of normalizing globally.
- For new notes, prefer the structures in `templates/` (`cheatsheet_template.md`, `reference_template.md`, `troubleshooting_template.md`, `howto_template.md`) and add the note under the relevant `docs/<topic>/` directory.
- If adding or renaming discoverable content, update the relevant root navigation note when it would otherwise become stale, especially `INDEX.md` or `TAGS.md`.

## Verification

- There is no automated verification configured; use focused Markdown review and `git diff --check` for whitespace/conflict-marker issues.
- Before finalizing, check `git status --short`; untracked Obsidian-generated files may exist and should not be modified or removed unless they are part of the requested change.
