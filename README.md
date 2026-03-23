# Modular Engineering Guides

A shared guide library of engineering principles, processes, and standards for AI-assisted development. Copy this library into any project to give GitHub Copilot (and other AI coding agents) the context they need to follow your team's quality standards.

---

## What's in This Repo

```
.github/
├── copilot-instructions.md          ← Repository-wide Copilot constitution (template)
├── instructions/
│   ├── testing.instructions.md      ← Path-scoped: test file standards
│   └── security.instructions.md    ← Path-scoped: security standards (all files)
├── docs/
│   ├── philosophy/                  ← 6 core engineering philosophy guides
│   ├── processes/                   ← 9 process guides (workflow, spike, UAT, etc.)
│   ├── standards/                   ← 3 code and test quality standards
│   └── examples/                    ← Real-world success stories and templates
└── skills/                          ← Reusable agent skill definitions
AGENTS.md                            ← GitHub Copilot Coding Agent instructions
```

---

## GitHub Copilot Instruction Layers

GitHub Copilot supports four instruction surfaces. Use them together for maximum coverage:

| Layer | File / Location | Scope |
|---|---|---|
| **Organization** | Org Settings → Copilot → Custom Instructions | All repos in the org |
| **Repository-wide** | `.github/copilot-instructions.md` | All Copilot interactions in this repo |
| **Coding Agent** | `AGENTS.md` (repo root) | Copilot Coding Agent tasks only |
| **Path-specific** | `.github/instructions/**/*.instructions.md` | Files matching the `applyTo` glob |

### How to Use These Files in Your Project

1. **Copy the constitution template** to your repo:
   ```
   cp .github/copilot-instructions.md /path/to/your-repo/.github/copilot-instructions.md
   ```
   Replace `[PROJECT_NAME]` and `[RATIFICATION_DATE]` with your project values.

2. **Copy the guide library** so Copilot can reference the full docs:
   ```
   cp -r .github/docs /path/to/your-repo/.github/docs
   ```

3. **Copy `AGENTS.md`** to your repo root for Coding Agent tasks:
   ```
   cp AGENTS.md /path/to/your-repo/AGENTS.md
   ```

4. **Copy path-specific instructions** if you want automatic scoping:
   ```
   cp -r .github/instructions /path/to/your-repo/.github/instructions
   ```

5. **Alternatively**, reference this repo from Copilot Chat using `@kdtix-open/modular-engineering-guides` or point at:
   `https://github.com/kdtix-open/modular-engineering-guides/tree/main/.github/docs`

---

## Guide Library Contents

### Philosophy (`.github/docs/philosophy/`)

| Guide | Summary |
|---|---|
| `celebrate-early-discovery.md` | Bugs caught early are quality wins, not failures |
| `tdd-principles.md` | Red → Green → Refactor — always |
| `proper-discovery.md` | Research before coding; spike when unfamiliar |
| `baseline-first-testing.md` | Record the baseline before any change |
| `independent-verification.md` | Fresh-eyes review before merge |
| `security-vulnerability-management.md` | Security is not a separate phase |

### Processes (`.github/docs/processes/`)

| Guide | Summary |
|---|---|
| `development-workflow.md` | The 9-phase development workflow |
| `spike-template.md` | Spike charter template |
| `moscow-prioritization-guide.md` | MoSCoW scope discipline |
| `environment-setup-guide.md` | Devcontainer and environment setup |
| `uat-testing-guide.md` | UAT scenario format and execution |
| `cross-platform-considerations.md` | OS-agnostic coding patterns |
| `documentation-tracking-guide.md` | Documentation and ticket tracking |
| `playwright-ui-ux-testing.md` | UI/UX and accessibility testing |
| `azure-ai-agent-guide.md` | Azure AI Agent patterns |

### Standards (`.github/docs/standards/`)

| Guide | Summary |
|---|---|
| `code-quality-standards.md` | Function size, complexity, naming |
| `testing-requirements.md` | Coverage minimums and test quality rules |
| `pre-commit-verification.md` | Phase 7 pre-commit checklist |

### Examples (`.github/docs/examples/`)

| File | Summary |
|---|---|
| `confluence-attachments-success-story.md` | Real-world example: 6 MCP tools, zero production bugs |
| `constitution-template.md` | Full Copilot constitution template (copy to your repo) |

---

## License

See [LICENSE](LICENSE).
