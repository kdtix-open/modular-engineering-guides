## Summary

This issue tracks the addition of GitHub Copilot layered instruction files and a project README
to the `modular-engineering-guides` repository. These changes adapt the existing Codex-style
guide library for GitHub Copilot's four-layer custom instruction system.

## Branch

`copilot/add-agents-md-file` → PR #1

## Changes Made

### New files added (760 lines across 6 files)

| File | Purpose |
|------|---------|
| `AGENTS.md` | GitHub Copilot Coding Agent entrypoint — guide library detection/bootstrap, Phase 7 pre-commit checklist, bug discovery protocol, guide library reference table |
| `README.md` | Repository documentation — explains the four Copilot instruction layers, repo structure, and adoption steps |
| `.github/copilot-instructions.md` | Repo-wide Copilot constitution template — full 8-principle constitution with dual detection paths (shell + chat/no-shell) |
| `.github/docs/examples/constitution-template.md` | Standalone constitution template for easy copy-paste into adopting repos |
| `.github/instructions/testing.instructions.md` | Path-scoped (`**/*.test.*`, `**/*.spec.*`, …) testing standards |
| `.github/instructions/security.instructions.md` | Path-scoped (`**/*`) non-negotiable security standards |

## Four-Layer Copilot Instruction Coverage

| Layer | File / Location | Scope |
|-------|----------------|-------|
| Org-wide | Org Settings → Copilot → Custom Instructions | All repos in org |
| Repo-wide | `.github/copilot-instructions.md` | This repo |
| Coding Agent | `AGENTS.md` | GitHub Copilot Coding Agent |
| Path-specific | `.github/instructions/*.instructions.md` | Per-file-pattern |

## Manifest Updates

Both `copilot-instructions.md` and `constitution-template.md` include two previously missing guide entries:
- `.github/docs/processes/azure-ai-agent-guide.md`
- `.github/docs/examples/confluence-attachments-success-story.md`

## Acceptance Criteria

- [x] `AGENTS.md` contains guide library detection block with bootstrap command
- [x] `AGENTS.md` has Phase 7 pre-commit checklist and bug discovery protocol
- [x] `.github/copilot-instructions.md` serves as repo-wide Copilot constitution template
- [x] Constitution template includes both Option A (shell) and Option B (no-shell) detection paths
- [x] Path-scoped instruction files created for testing and security standards
- [x] `README.md` documents all four Copilot instruction layers
- [x] All guide manifests updated with complete file list
