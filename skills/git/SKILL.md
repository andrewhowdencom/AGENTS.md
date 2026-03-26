---
name: git
description: Guidelines for validation, staging, amending, and commit messages.
---

# Git

## Validate Changes
Before committing any changes, ensure that your work does not break the application and meets all quality standards.
Run the `validate` task just prior to commit:

```bash
task validate
```

## Stage Changes
When staging changes, stage only the files that you have changed as part of your current stream of work

```bash
git add ./path/to/file ./path/to/file2
```

⚠️ Never stage changes by adding the whole directory:

```bash
git add .
```

Instead, if you wish to stage folders, stage those also deliberately:

```bash
git add path/to/folder
```

## Amend Commits
If you need to update the previous commit (e.g., to squash changes or fix a mistake), append changes to the last commit:

```bash
git commit --amend
```

If you have already pushed, you will need to force push:
```bash
git push --force-with-lease
```

## Write Commit Messages
Commit messages should follow the standard format:

#### Title
Maximally 72 characters, descriptive, using imperative mood (e.g., "Deploy changes automatically" not "Deployed changes").

#### Body
Maximally 72 characters, Explain **why** the change was made (justification), not just **what** changed. Use headers for structured context if applicable:

```
Design:
  Describe the design of the change here.

Tradeoffs:
  Explain any tradeoffs (performance, complexity, etc).

Justification:
  Provide the reasoning for this change.
```

#### Co-author
If pair programming or crediting others, add co-authors at the end of the body:

```
Co-authored-by: NAME <NAME@EXAMPLE.COM>
```

Coding agents (e.g. Gemini, Claude, Antigravity) are all pair-programmers, and should ensure their own emails are added in "Co-Authored-By".
