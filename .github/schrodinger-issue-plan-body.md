## 🧠 Summary

**Schrödinger Issue** is a GitHub workflow skill that retroactively creates an issue from local
changes—making it appear as though the work was tracked from the start.

> *"Turns local changes into a 'Schrödinger issue'—an issue that doesn't exist until observed, then appears as if it always did."*

## 🎯 Problem / Motivation

Developers often:
- Make meaningful progress before opening an issue
- Forget to track exploratory work, failed attempts, or evolving intent
- Lose valuable context that explains *why* changes were made

This skill solves that by:
- Analyzing current local changes (diffs, commits, uncommitted work)
- Generating a structured GitHub issue
- Capturing intent, experiments, outcomes, and remaining work
- Enabling commits to reference an issue that now "exists"

## �� What Was Tried

The skill system (`skill-creator`) was used to scaffold the skill following the standard four-step
lifecycle (understand → plan → init → edit → validate). The bundled `scripts/create_issue.py`
collects git context and creates the issue via `gh issue create --body-file`, matching the
`gh`-first pattern introduced in the `copilot-setup-steps.yml` workflow.

## ✅ What Worked

A minimal, well-structured skill was created:
- `SKILL.md` — concise workflow instructions (106 lines) covering prereqs, context collection,
  analysis guidance, issue creation, and return value
- `scripts/create_issue.py` — deterministic script that runs `--dry-run` to dump context JSON,
  then creates the issue via `gh CLI` with graceful label fallback
- `agents/openai.yaml` — UI metadata with correct `display_name`, `short_description`,
  and `default_prompt`

## 📦 Changes Made

| File | Description |
|------|-------------|
| `.github/skills/schrodinger-issue/SKILL.md` | Skill instructions and workflow |
| `.github/skills/schrodinger-issue/scripts/create_issue.py` | Issue creation script (177 lines) |
| `.github/skills/schrodinger-issue/agents/openai.yaml` | UI metadata |
| `.github/schrodinger-issue-plan-body.md` | This issue body (used as `--body-file`) |
| `.github/workflows/create-schrodinger-issue.yml` | `workflow_dispatch` to create this issue post-merge |

## ⚠️ Remaining Work

- [ ] Add label auto-suggestion based on diff heuristics (bug vs enhancement vs refactor)
- [ ] Detect test coverage gaps and mention in Remaining Work section
- [ ] Multi-commit narrative stitching for long-lived branches
- [ ] Optional pre-commit / pre-push hook integration
- [ ] Auto-link generated issue to open PR (`gh pr edit --body "Closes #N\n..."`)
- [ ] Confidence scoring for inferred intent when diff is ambiguous

## 🧾 Notes / Learnings

- The `init_skill.py` scaffolding script is the canonical way to create new skills; it handles
  `SKILL.md` templating, `agents/openai.yaml` generation, and resource directory creation.
- Skills must pass `quick_validate.py` before committing (frontmatter name, description, etc.)
- The sandbox's `GITHUB_TOKEN` is read-only at session start; `workflow_dispatch` from `main`
  post-merge is the correct pattern for write operations (same as `create-issue-for-agents-md.yml`).
- Progressive disclosure matters: SKILL.md body stays under 500 lines; heavy logic lives in scripts.

## 🔗 Traceability

- Branch: `copilot/add-agents-md-file-again`
- PR: https://github.com/kdtix-open/modular-engineering-guides/pull/6
- Skill created using: `.github/skills/.system/skill-creator/scripts/init_skill.py`
- Validated with: `.github/skills/.system/skill-creator/scripts/quick_validate.py`
