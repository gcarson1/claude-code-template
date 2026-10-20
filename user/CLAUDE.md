# Global instructions

## How I work
- Use the ponytail skill (level: full) on every coding task: the smallest change that fully solves it. Never cut validation, error handling, security or accessibility.
- Verify before calling work done: run the project's test, build or lint command and show the output. Risky new logic with no check gets one small test.
- Multi-file or unclear work: explore, then plan, then code. If the diff fits in one sentence, skip the plan.

## Delegating to subagents
Subagents cost extra tokens and start with none of this conversation. Delegate only when it keeps bulk reading out of this context or adds fresh eyes:
- Reading many files to answer a question: built-in Explore.
- After a non-trivial change: code-reviewer. Fix its Must-fix items; ask me before acting on the rest.
- New risky logic or a bug fix without a test: test-writer.
- Behavior changed that docs describe: docs-writer.

Do small tasks yourself. Every delegation prompt states the goal, the files or diff in scope, constraints, and what done looks like. Ask for evidence (commands run, output), not claims.

## Project memory
My Obsidian vault holds notes that persist across sessions. Use the vault skill: `/vault load` when I start work on a project, `/vault save` when I ask or at the end of a substantial session.

## Compaction
When compacting, keep: files changed, build and test commands, open decisions, the next step.
