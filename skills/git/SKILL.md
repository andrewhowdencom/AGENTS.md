---
name: git
description: >-
  Guidelines for validation, staging, amending, and commit messages.
  Load this skill whenever you are about to run git add, git commit,
  or git push.
---

# Git

When working with Git, follow these rules. Violating any of them
produces commits that fail project standards.

## When To Apply This Skill

Load this skill and follow these rules whenever you are about to:

- Stage files (`git add`)
- Create a commit (`git commit`)
- Amend a commit (`git commit --amend`)
- Push commits (`git push`)

If you have already committed without following these rules,
amend the commit before pushing.

## 1. Validate Before Committing

Before committing, verify that your changes do not break the
project and are tidy. Run whatever validation the project provides:

- If the project has a build or test command, run it (e.g.,
  `make test`, `go test ./...`, `npm test`, `cargo test`).
- If the project has a linter or formatter, run it and fix any
  issues it reports.
- If you created temporary files, debug output, or commented-out
  code during development, remove them before staging.

Do not commit changes that fail validation. Fix issues first,
then re-validate.

## 2. Stage Deliberately

Stage only the files you have changed as part of your current
stream of work. Never use `git add .` — it stages everything,
including files you did not intend to commit.

Correct:

```bash
git add path/to/file.go path/to/file_test.go
```

Also correct (staging a whole deliberate folder):

```bash
git add path/to/folder/
```

Wrong — do not do this:

```bash
git add .
```

After staging, review what you are about to commit:

```bash
git diff --staged --stat
git diff --staged
```

Confirm that only intended changes are staged. If you see
unrelated changes, unstage them with `git reset HEAD <file>`
and re-stage correctly.

## 3. Amend When Fixing

If you need to update your most recent commit (to fix a mistake,
add a missing file, or improve the message), amend it rather
than creating a new commit:

```bash
git commit --amend
```

If you have already pushed the commit, force-push with lease:

```bash
git push --force-with-lease
```

## 4. Write Commit Messages

Every commit must have a title and a body. Both are required.

### Title

- Maximum 72 characters.
- Use imperative mood: "Add validation step" not "Added
  validation step" or "Adds validation step".
- Summarize what the change does at a high level.

### Body

- Maximum 72 characters per line.
- Explain **why** the change was made, not just what changed.
- Structure the body using these headers:

```
Design:
  Describe the technical approach — how the change is
  implemented and why that approach was chosen.

Tradeoffs:
  Note any tradeoffs: performance costs, added complexity,
  alternative approaches considered and rejected.

Justification:
  State why this change is needed — the problem it solves
  or the improvement it delivers.
```

Example of a well-formed commit message:

```
Add pre-commit validation guide to git skill

Design:
  Replace the project-specific "task validate" instruction
  with generic validation guidance that works in any
  project. The new text describes checking build, tests,
  linters, and removing temporary artifacts.

Tradeoffs:
  The generic guidance is less precise than a concrete
  command, but ensures the skill is usable across all
  projects rather than only those using Taskfile.

Justification:
  The previous instruction referenced a tool that does not
  exist in most repositories, causing agents to skip the
  entire validation step. Generic guidance ensures the
  validation habit is established regardless of tooling.
```

### Co-authored-by

When an AI coding agent (Claude, Gemini, Copilot, etc.)
assisted in producing the commit, add a `Co-authored-by`
trailer at the end of the body. Use the agent's known
identity. If the agent's email is not known, use a
placeholder consistent with the project's conventions.

For example:

```
Co-authored-by: Claude <claude@anthropic.com>
```