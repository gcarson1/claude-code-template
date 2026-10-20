# Claude Code template

A general setup for any project: three lean subagents, Ponytail as the coding standard, an Obsidian vault as memory across sessions, and Gemini as an optional second model.

## What goes where

```
user/      → ~/.claude/          installed once, applies to every project
  CLAUDE.md                      how Claude works + when to delegate
  agents/                        code-reviewer, test-writer, docs-writer
  skills/                        ponytail, ponytail-review, ponytail-audit, ponytail-debt, vault
  settings.snippet.json          gives Claude access to the vault folder
project/   → each new repo       copied per project
  CLAUDE.md                      skeleton: commands, conventions, gotchas
  .mcp.json                      Gemini MCP server (key read from env)
  .gitignore                     keeps secrets and personal files out of git
vault/     → ~/Documents/Vault   open this folder in Obsidian
```

Why user-level: subagents and skills in `~/.claude/` work in every project with no copying, and you update them in one place. A project can still override one by putting a file with the same name in its own `.claude/agents/`.

## Get it

```bash
git clone https://github.com/gcarson1/claude-code-template.git
```

Or download [`claude-code-template.zip`](claude-code-template.zip) from this repo and unzip it. The green **Code → Download ZIP** button on GitHub always gives the very latest version.

## Install (once)

```bash
cd claude-code-template
mkdir -p ~/.claude/agents ~/.claude/skills
cp -n user/agents/*.md ~/.claude/agents/
cp -Rn user/skills/* ~/.claude/skills/
cp -n user/CLAUDE.md ~/.claude/CLAUDE.md   # skipped if you already have one: merge by hand
cp -Rn vault ~/Documents/Vault             # only if that folder doesn't exist yet
```

`-n` never overwrites a file you already have. (On Linux, `cp` prints a harmless warning about `-n`.)

Then:
1. Add `"additionalDirectories": ["~/Documents/Vault"]` under `"permissions"` in `~/.claude/settings.json`. Create the file from `user/settings.snippet.json` if it doesn't exist. If the vault lives elsewhere, also change the path on the `Vault:` line of `~/.claude/skills/vault/SKILL.md`.
2. Open `~/Documents/Vault` in Obsidian (Open folder as vault).
3. For Gemini, set `GEMINI_API_KEY` in your shell profile (steps in `gemini-setup.md`).
4. Check: start `claude` anywhere and run `/agents` (three agents listed) and `/skills` (ponytail, vault, and the rest).

## New project (each time)

```bash
cp -n path/to/claude-code-template/project/{CLAUDE.md,.mcp.json,.gitignore} .
```

Fill in `CLAUDE.md`, or run `/init` and keep only what Claude can't work out from the code. Approve the `gemini` server the first time `claude` asks.

## How the subagents are built

Each subagent is a markdown file with YAML front matter (`name`, `description`, `tools`, `model`, `skills`), followed by its system prompt. Every rule in them comes from Anthropic's guidance, Ponytail, or both:

| Rule | Anthropic says | Ponytail says |
|---|---|---|
| Few agents, each with one job; built-in Explore and Plan cover research | Design focused subagents. Multi-agent runs use ~15× the tokens of chat, and coding has few truly parallel tasks | "Does it need to exist?" Reuse what is already there |
| The reviewer reuses Ponytail's review skill instead of writing its own | Preload skills with the `skills:` field | Reuse beats writing new code |
| Every agent preloads Ponytail | A subagent doesn't see your conversation or invoked skills; restate the rules it must follow | Ponytail's plugin injects its rules into subagents for the same reason |
| Least tools: the reviewer can't edit, the docs-writer has no shell | Limit tool access for security and focus | Smallest surface that works |
| Description = when to use + what comes back, with "use proactively" | Claude routes on the description; make it single out one agent | Short and plain |
| Every prompt starts "you see none of the conversation" and says where to look (git diff, the repo) | Each delegation needs an objective, output format, tools and boundaries | Read the code first; scope comes from the code, not guesses |
| Findings need a concrete failing case; "Ship" is a valid result | Reviewers asked to find gaps over-report, which leads to over-engineering. Flag only correctness and requirement gaps | "No case, no finding. No style taste, no 'consider'." |
| One good test per real risk; show a bug-fix test failing first | Give Claude a check it can run; ask for evidence, not claims | Risky logic leaves one small test, not coverage |
| Reports are capped (≈200–1,500 words) with a fixed shape | Subagents should return a condensed summary (often 1,000–2,000 tokens) | A reply a busy human reads in one go; say what wasn't checked |
| Plain instructions, no wall of MUSTs | Right altitude: clear heuristics, not brittle rules. Emphasis on one line only | Short sentences, everyday words |
| `shortcut:` comment in docs-writer's front matter | Prompts are advisory; hooks are the hard guarantee | Mark a deliberate shortcut with its limit and when to upgrade it |

**Models.** The reviewer uses `inherit`, so it gets your session's model, because review quality matters most. Tests and docs use `sonnet` to save usage. Change the `model:` line to `haiku`, `opus` or `inherit` to trade cost against quality.

**Limits, stated plainly.** The `tools` field can't restrict paths. The docs-writer stays in markdown because its prompt says so; the reviewer could technically run a writing shell command but is told not to. If either ever strays, add a `PreToolUse` hook (Anthropic's subagent docs show an example).

### Make your own subagent

```markdown
---
name: kebab-name
description: <What it does>. Use proactively when <trigger>. Returns <shape>.
tools: Read, Grep, Glob            # add only what the job needs
model: sonnet
skills:
  - ponytail
---

You <role> in a repo you did not write. You see none of the conversation: work from the task prompt and <where to look>.

## Scope
<What it may touch, and what it never does.>

## Process
<3–5 steps, as heuristics.>

## Report (under about N words)
<Fixed shape. End with what was not checked.>
```

Before adding one, ask: would Explore, Plan, or just the main session do this fine? If yes, skip it.

## Using it

**Automatic:** describe the work and Claude routes it. For example, after "Add rate limiting to the API", the global `CLAUDE.md` tells Claude to call code-reviewer, and test-writer if risky logic is new.

**Explicit:** name the agent, or type `@` and pick it. For example: "Use code-reviewer on the last commit against SPEC.md" or "@test-writer cover `parse_invoice`". `claude --agent code-reviewer` runs a whole session as that agent.

**Ponytail:** it is on for coding tasks via the global `CLAUDE.md`.
- `/ponytail ultra` makes it push back on the request itself; `/ponytail lite` turns it down.
- `/ponytail-audit` reviews the whole repo; `/ponytail-debt` lists every `shortcut:`.
- Prefer the full plugin (status band, level that persists)? Install it with `/plugin marketplace add DietrichGebert/ponytail`, then `/plugin install ponytail@ponytail`, and delete the four `~/.claude/skills/ponytail*` folders so the rules aren't loaded twice.

**Vault:** say `/vault load` at the start of a session, `/vault save` at the end, and `/vault decide` when you make a call you'll want to remember. Claude reads the project's `_index.md` and the latest log entries, so a new session picks up in seconds.

**Gemini:** say "Ask Gemini for a second opinion on this design" or "Use Gemini to summarize this long file". Don't send anything confidential unless your data policy allows it.

## Vault vs Google Drive (MCP)

- **Drive** is for documents that live elsewhere and change: specs others edit, shared spreadsheets. Read them live when the current version matters.
- **The vault** is for your own running context: what happened, what's next, what you decided and why. It is structured, local, private and fast to load.
- **Typical loop:** read the source from Drive, then `/vault save` the takeaways, so the next session never needs to re-read the original.

## Updating Ponytail

The copied skills are pinned to commit `09c434a`. To update, clone the repo again and copy `skills/{ponytail,ponytail-review,ponytail-audit,ponytail-debt}/SKILL.md` over the copies in `~/.claude/skills/`.

## Sources
- [Claude Code: subagents](https://code.claude.com/docs/en/sub-agents)
- [Claude Code: best practices](https://code.claude.com/docs/en/best-practices)
- [Claude Code: memory / CLAUDE.md](https://code.claude.com/docs/en/memory)
- [Claude Code: permissions](https://code.claude.com/docs/en/permissions)
- [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Anthropic: Effective context engineering for AI agents](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Ponytail (MIT)](https://github.com/DietrichGebert/ponytail)
