---
name: vault
description: Load or save project memory in my Obsidian vault, markdown notes that persist across sessions. Use for "/vault load", "/vault save", "/vault decide", "where did we leave off", "save this to my notes", "log this decision", or when starting work on a project that has notes.
argument-hint: "[load|save|decide] [project]"
---

Vault: `~/Documents/Vault` (edit this line if yours lives elsewhere).
Project: the name given, else `Vault project:` in the repo's CLAUDE.md, else the repo folder name, in lowercase-kebab-case.

## Layout
- `00-Inbox/` unsorted captures
- `10-Projects/<project>/`
  - `_index.md` what it is, status, links, next steps (from `90-Templates/project.md`)
  - `log.md` session log, newest entry first
  - `decisions.md` decision records, newest first (format in `90-Templates/decision.md`)
  - anything else as `YYYY-MM-DD-topic.md` (meeting notes from `90-Templates/meeting.md`)
- `20-Knowledge/` lessons that apply beyond one project, one topic per file
- `90-Templates/` copy these; never edit them in place

Every note starts with front matter: `type`, `project`, `date` (YYYY-MM-DD), `status`, `tags`. File names are lowercase-kebab-case. Link notes with `[[wikilinks]]`.

## load
Read `_index.md` and the top three entries of `log.md`. Reply in five lines at most: status, last session, next step, open questions. Read other notes only when the task needs them. No folder for this project: say so and offer to create one from the template.

## save
Prepend one entry to `log.md`:

```
### YYYY-MM-DD
- Done: what changed this session (files, features, fixes)
- Next: the next concrete step
- Open: unresolved questions or risks
```

Then update `_index.md` if status or next steps changed. Only facts from this session; mark guesses `(inferred)`. Keep the entry under ten lines.

## decide
Prepend a record to `decisions.md` using `90-Templates/decision.md`: the decision, why, what was rejected, and when to revisit.

## Rules
- Edit or append; never delete or rename notes without asking.
- No secrets, credentials or tokens in the vault.
- Keep notes short: they are reloaded into context later, and every line costs tokens.
