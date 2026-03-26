# Copilot Instructions

## What This Repository Is

A content-only repository with two deliverables:

1. **`Agents.md`** — A dual-purpose file that functions as both:
   - The `AGENTS.md` for this repo (guides AI agents working here)
   - A **template** for creating `.github/instructions/copilot-instructions.md` in target projects
   - The split point is `<!-- END TEMPLATE META-INSTRUCTIONS -->`: everything before is meta-instructions, everything after is template content

2. **`.github/skills/`** — A library of Copilot CLI skills, each a self-contained folder

3. **`.github/docs/`** — Modular engineering guides (not skills; referenced by the Agents.md template)

There is no build system, no tests, and no linter. All content is Markdown and YAML.

---

## Skill Anatomy

Every skill lives at `.github/skills/<skill-name>/` and follows this structure:

```
skill-name/
├── SKILL.md                  ← required; triggers the skill
│   ├── YAML frontmatter      ← name + description (what Copilot reads to decide when to use it)
│   └── Markdown body         ← instructions loaded only after the skill triggers
├── agents/
│   └── openai.yaml           ← UI metadata (display_name, icon, default_prompt)
├── references/               ← docs loaded into context on demand (not always)
├── scripts/                  ← executable helpers (Python/Bash)
└── assets/                   ← icons, templates, fonts used in output
```

**Critical**: The `description` in SKILL.md frontmatter is the sole trigger signal. Make it precise about when to use — and when NOT to use — the skill.

**System skills** live at `.github/skills/.system/` (`skill-creator`, `skill-installer`, `openai-docs`). These are pre-installed and should not be modified without care.

---

## Docs Structure

```
.github/docs/
├── philosophy/    ← core principles (tdd-principles, celebrate-early-discovery, etc.)
├── processes/     ← step-by-step workflows (development-workflow, uat-testing-guide, etc.)
├── standards/     ← quality requirements (code-quality-standards, testing-requirements, etc.)
└── examples/      ← real-world case studies
```

These files are referenced by the Agents.md template but live here in this repo.

---

## Key Conventions

- **Skill SKILL.md frontmatter must have `name` and `description`** — these are the only fields Copilot reads to decide whether to invoke a skill. Keep `description` comprehensive and boundary-setting.
- **`agents/openai.yaml`** is optional but enables richer UI in Copilot chat (display name, icon, chip).
- **`references/`** files are loaded on-demand during skill execution — they should be laser-focused docs, not general background.
- **Agents.md is a template, not just instructions** — when applying it to a target project, only content after `<!-- END TEMPLATE META-INSTRUCTIONS -->` belongs in the output file.
- **VS Code auto-approves `git push`** (configured in `.vscode/settings.json`).
