# Multi-Sub-Agent Plan Build — Canonical Stages 0–8

> **Status:** RATIFIED (kdtix-open / Chris Kreager, 2026-04-29)
> **Audience:** Any AI agent (Claude Code / Codex / Copilot / Cursor) authoring a KDTIX-format plan markdown for `/plan-to-project` consumption.
> **Authority:** This is a **MUST-FOLLOW** process for every plan created via the multi-sub-agent build pattern. Cross-referenced from `.github/copilot-instructions.md`, project-root `CLAUDE.md`, and `docs/plans/Plan-of-Plans.md`. Skipping or collapsing stages produces shallow, incomplete plans that the orchestrator cannot dispatch reliably.

---

## Why this exists — the operator's directive (2026-04-29)

> *"I want full depth details so that every /plan-to-project created {project scope|initiative|epic|user story|task} has all template [MoSCoW | dependencies | assumptions | success criteria | artifacts | I know when I am done | Security/Compliance | `As a [ROLE], I want [WHAT], So that [OUTCOME]` | Why This Matters | mermaid diagrams] are all filled out by using multiple agents to leverage as much free context windows as needed to build rich deep documented issues."*

Single-agent plan authoring fails for backlogs spanning multiple Epics × multiple Stories per Epic. Empirical evidence:

- Single-agent runs against >50-issue backlogs hit context-window limits. The agent enters shallow mode (placeholder content like `_TBD_` / `[ITEM]` / `[ROLE]`) or stream-times-out (17–36 minutes with zero issue writes was observed during Slice 2 Round 1 attempts).
- Per-Epic sub-agent fan-out keeps each prompt under a comfortable context budget. 6 sub-agents × ~5 stories each = 30 stories enriched in two ~20-minute parallel batches with no agent timeouts (Slice 2 PRs #567–#572 and #574–#579 prove this).
- Audit + depth-fill stages catch the systematic gaps that single-pass authoring leaves behind (e.g. half the Epics under EP-OH-* in the Helm plan lacked Mermaid diagrams in their first pass; a second per-Epic depth-fill closed every gap).

The 9-stage taxonomy below codifies this pattern as a repeatable, multi-agent process.

---

## Stage taxonomy

| # | Name | Owner | Output | Parallelism |
|---|---|---|---|---|
| 0 | Recon (Context Map) | 1 recon sub-agent | `/tmp/<plan-slug>-context-map.md` | n/a |
| 1 | Architect (Hierarchy Skeleton) | Architect (Opus 4.7 1M for context-heavy plans; Sonnet for smaller scopes) | base plan markdown w/ Scope + Initiative + Epic stubs | n/a |
| 2 | Per-Epic Authoring Fan-Out | N sub-agents (1 per Epic) | N PRs against base branch (Epic body + Stories + Stage-2) | parallel |
| 3 | Full-Depth Audit | 1 audit sub-agent | `/tmp/<plan-slug>-audit-report.md` | n/a |
| 4 | Per-Epic Depth-Fill Fan-Out | N sub-agents (1 per Epic) | N depth-fill PRs (Mermaid + Artifacts + Security/Compliance + heading normalization) | parallel |
| 5 | Integration & Validation | Architect | base branch consolidated; FR #45 schema-gate AND `mmdc` Mermaid batch validation BOTH pass (zero parse errors) | n/a |
| 6 | Backlog Seeding | 1 driver process | issues created in target GitHub Project; manifest archived | n/a |
| 7 | Operator Review + Refresh Loop | Operator + 1+ refresh sub-agents | spot-check report; targeted refresh PRs if gaps surface | iterative |
| 8 | Pattern Capture & Continuous Improvement | Architect + skill maintainer | issues filed; lessons captured in `references/`; `CLAUDE.md` gotchas updated | n/a |

Stages 0–6 are MUST for every new plan. Stages 7–8 are MUST for iterating an existing plan or improving the process for the next plan.

---

## Stage 0 — Recon (Context Map)

**Purpose.** Synthesize the source documents (existing plans, License Boundary Catalog, related Project Scopes, prior architectural decisions) into a tight context map the Architect uses to anchor design choices.

**Process.**
1. Architect identifies the source docs (typically: 1 plan-source markdown, License_Boundary_Catalog.md, related plans the new plan must coordinate with, the operator's directive that triggered the plan).
2. Dispatch ONE recon sub-agent with explicit reading list + structured output specification.
3. Recon agent produces a markdown report at `/tmp/<plan-slug>-context-map.md` (≤8 KB, dense, dispatchable).
4. Recon report sections (in this order):
   - **Established Commitments** — verbatim quotes of license language, audience tiers, hosting, non-negotiables. Architect MUST honor these.
   - **License Boundary Catalog — where this scope sits** — domain-by-domain table with Row #, name, license zone (BSL / MIT / Mixed / Subscription), and how this plan interacts.
   - **Self-Healing R-Rules / autonomy contract** — which R-rules apply.
   - **Status of any in-flight makeshift implementation** that this plan replaces.
   - **Pattern reuse candidates** (e.g. Bridge Slice 2 npx-installer pattern, OIDC device-flow middleware).
   - **Existing adjacent plans + cross-references**.
   - **The Architect's straw-man Epic list** (annotated with alignment / gaps / dependencies per Epic).
   - **Open questions** for the Architect to decide.

**MUST.**
- Recon agent reads the FULL primary source doc (no skimming).
- Quote license language verbatim where it matters; paraphrase elsewhere.
- Make NO design recommendations beyond "alignment / dependencies" annotations — the Architect makes decisions, not the recon agent.

**MUST NOT.**
- Pre-author Epic content.
- Propose implementation details (file paths, code, library choices).

---

## Stage 1 — Architect (Hierarchy Skeleton)

**Purpose.** Author the base plan markdown's foundational structure — Project Scope + Initiative + Epic stubs — with explicit design decisions locked in.

**Process.**
1. Architect (Claude Opus 4.7 1M context for plans referencing >30 KB of source docs; Sonnet 4.x for smaller plans) reads the Stage-0 context map.
2. Architect surfaces design decisions to the operator (license zone, state store choice, event-bus implementation, OIDC reuse vs separate registration, etc.) and gets explicit approval for non-trivial calls.
3. Architect authors the base file with:
   - **Project Scope** — Vision, Business Problem & Current State, Success Criteria, In-Scope Capabilities, Assumptions, Out of Scope, MoSCoW Classification, I Know I Am Done When, Initiatives table, **Architecture & Diagrams** (≥1 Mermaid C4 Context).
   - **Initiative** — Objective, Release Value, Success Criteria, Feature Scope, Architecture & Diagrams (≥1 Mermaid C4 Container or flowchart), Assumptions, Dependencies table, I Know I Am Done When, Epics table.
   - **N Epic stubs** — heading + 1–2 sentence description + Blocking declaration. **No content beyond the stub at this stage.** The fan-out agents fill Epic content.
4. Architect commits the base file to a feature branch (e.g. `plan/<scope-slug>-<phase-slug>-foundation`) and pushes.
5. Architect documents the locked design decisions in the commit message body so subsequent agents (and the operator on review) see the trail.

**MUST.**
- Re-parenting plan recorded for any existing seeded GitHub issues that this plan absorbs (operator runs `kdtix-open/skill-plan-to-project#33` amend mode after the plan ships).
- License zone declared on every Epic stub (BSL 1.1 / MIT / Mixed / subscription).
- Phase boundaries explicit if the plan is multi-phase (Phase 1 = MUST / Phase 2 = MUST / Phase 3 = COULD typical).
- Mermaid diagrams at Project-Scope and Initiative levels (≥1 each).

**MUST NOT.**
- Pre-author Epic body content (Objective / Release Value / Success Criteria / Done-When for Epics) — that's Stage 2.
- Skip the design-decisions surface-to-operator step for non-trivial calls.

---

## Stage 2 — Per-Epic Authoring Fan-Out

**Purpose.** Each Epic's content is authored by its OWN sub-agent in its own git worktree. Per-Epic context-window scoping prevents shallow output across all Epics.

**Process.**
1. Architect creates N git worktrees (one per Epic): `git worktree add /tmp/<plan-slug>-ep-<NNN>-content origin/<base-branch>` (or `git clone --branch <base-branch>` for cross-repo plans).
2. Architect dispatches N sub-agents in parallel (1 per Epic). Each agent's prompt MUST include:
   - **Worktree path + branch name** (`plan/<plan-slug>-ep-<NNN>-content`)
   - **Single file to edit** (the base plan markdown)
   - **Auth** (App-token mint command for org access)
   - **Required reading list** (Stage-0 context map + base plan file + relevant `operations-helm.md` or other source-doc Initiative section + 1–2 quality-benchmark issues from a prior plan)
   - **Stop boundary** ("edit only between `### Epic: <ID>` and the next `### Epic:` heading or end-of-file")
   - **Story list** (operator-stated; do NOT let the agent invent random stories)
   - **Per-Story Stage-2 format requirements** (User Story fenced code / MoSCoW Must/Should/Could/Won't / Assumptions Roles+Starting+Preconditions / Why-This-Matters 2–4 sentences / I-Know-I-Am-Done-When 6–9 testable bullets ending TDD / Acceptance Criteria 3–5 G/W/T scenarios with concrete values)
   - **License zone confirmation** per Story
   - **Quality benchmark** (cite a specific exemplar Story from a prior plan)
3. Each sub-agent edits its Epic stub block, opens a PR back to the base branch.
4. Architect monitors completions and surfaces PRs to the operator as they land.

**MUST.**
- Single-file edit per agent (the plan markdown). No cross-file edits.
- Single PR per agent. No grouped commits.
- Won't-Have rows reference sibling Stories explicitly ("that's Story OH-XXX-SY") — uses bare text, no link syntax.
- Concrete values in Acceptance Criteria (real route paths, real type names, real exit codes) — never `[ACTION]` / `[OUTCOME]` placeholders.

**MUST NOT.**
- Touch any other Epic's content.
- Modify Project Scope or Initiative content.
- Invent Stories outside the operator's list.
- Skip the per-Story Stage-2 subsection set.

---

## Stage 3 — Full-Depth Audit

**Purpose.** Walk the consolidated plan after Stage 2 PRs land. Produce a per-item gap report against the canonical full-depth subsection checklist. Stage 4 enrichment agents use this report as their precise input.

**Process.**
1. Architect admin-merges all Stage-2 PRs into the base branch (squash + delete-branch + admin to bypass status checks on doc-only PRs).
2. Architect dispatches ONE audit sub-agent with:
   - The plan file path
   - The canonical full-depth subsection checklist (verbatim — see "Canonical Full-Depth Subsection Checklist" below)
   - Output specification (markdown table with line ranges per item)
3. Audit agent walks every item (Scope / Initiative / Epic / Story / Task), produces:
   - **Per-item gap row** — Item ID, Type, Heading Format (exact line), Line Range, Subsections Present, Subsections MISSING, Mermaid Present (yes/no), Heading Normalization Needed (yes/no with canonical form)
   - **Summary section** — total items by type; heading-style variants with counts; per-Epic Story counts; systemic gap patterns; top-5 highest-impact gaps to address.
4. Audit report saved to `/tmp/<plan-slug>-audit-report.md` (≤25 KB).

**MUST.**
- Heading-style variants are caught and normalized in Stage 4 (canonical: `### Epic:` for Epics; `#### Story:` for Stories; `##### Task:` for Tasks).
- Mermaid coverage measured per item, not per Epic.
- Artifacts-table coverage measured per item.

**MUST NOT.**
- Pre-author missing subsections (the audit DESCRIBES gaps; Stage 4 FILLS them).
- Skip line-range references (Stage 4 agents jump directly to sections).

---

## Stage 4 — Per-Epic Depth-Fill Fan-Out

**Purpose.** Same fan-out pattern as Stage 2, but for ENRICHMENT. Each per-Epic agent fills the gaps the Stage-3 audit identified for its Epic.

**Process.**
1. Architect dispatches N depth-fill sub-agents (one per Epic) in parallel, each fed:
   - Their Epic's specific gap list from the audit report
   - The canonical full-depth subsection checklist
   - **Mermaid-diagram-per-item guidance**:
     - Sequence diagram for I/O flow Stories (request/response, event chain)
     - State diagram for state-machine Stories (lifecycle, status transitions)
     - Flowchart for orchestration Stories
     - C4 Context / Container / Component for architectural Stories
     - Use `mermaid` skill (kdtix-open user skill) to validate syntax before embedding if available
   - **Artifacts-table conventions** — deliverables: files, types, tests, docs, configs, Kafka topics produced/consumed
   - **Security/Compliance depth conventions** — input validation, no secrets, least-privilege, plus per-feature security notes (e.g. "OIDC scope claim verified before action dispatch", "no audio bytes retained in voice channel")
2. Each agent edits its Epic block (Mermaid blocks, Artifacts tables, full Security/Compliance, heading normalization) and opens a depth-fill PR.
3. Architect admin-merges depth-fill PRs sequentially (or in parallel where line ranges don't overlap).

**MUST.**
- Every Story has ≥1 Mermaid diagram fit-to-purpose (sequence / state / flowchart / C4) by end of Stage 4.
- Every Story has an Artifacts table.
- Every Story has a per-feature Security/Compliance section (NOT just the generic input-validation checklist).
- Heading style normalized to canonical (`### Epic:` / `#### Story:` / `##### Task:`).

**MUST NOT.**
- Touch other Epics' content.
- Modify Stage-2 content unless the audit explicitly flagged it.
- Replace existing rich content with shallower content.

---

## Stage 5 — Integration & Validation

**Purpose.** Verify the consolidated plan passes `/plan-to-project`'s schema gate AND every Mermaid diagram parses cleanly before backlog seeding.

**Process.**
1. Architect admin-merges all Stage-4 depth-fill PRs into the base branch.
2. Architect runs schema-gate dry-run:
   ```bash
   cd ~/.claude/skills/plan-to-project
   python3 -m scripts.create_issues refresh \
     --plan "<absolute-path-to-base-branch-checkout>/docs/plans/kdtix-format/<Plan-Name>.md" \
     --repo <ORG/REPO> \
     --scope-issue <SCOPE-NUM-OR-OMIT-FOR-CREATE> \
     --dry-run
   ```
3. If FR #45 schema gate FAILs (missing required subsections per the schema), drop back to Stage 4 with a focused fix-up agent for the offending item(s).
4. **Architect runs Mermaid syntax batch validation** across every `\`\`\`mermaid` block in the consolidated plan:
   ```bash
   # Extract all mermaid blocks to /tmp/<plan-slug>-mermaid-blocks/ then loop:
   for f in /tmp/<plan-slug>-mermaid-blocks/block-*.mmd; do
     mmdc -i "$f" -o "${f%.mmd}.svg" -q || echo "FAIL: $f"
   done
   ```
   Equivalent canonical Python harness shipped at `scripts/validate_plan_mermaid.py` (see References). MUST exit 0 for every block. Reference render: `mmdc` is the official Mermaid CLI (`@mermaid-js/mermaid-cli`); the `validate_and_render_mermaid_diagram` MCP tool is an acceptable substitute when MCP is the only available transport.
5. If any diagram FAILs parse:
   - Common causes (operator-observed across last several plan refreshes):
     - Curly braces / parens in flowchart node labels (wrap label in quotes: `["text (with parens)"]`).
     - Curly braces in flowchart edge labels (wrap: `-->|"{ok:true}"|`).
     - Semicolons (`;`) in sequenceDiagram messages or participant aliases (treated as statement separator → use commas instead).
     - Reserved keywords in sequenceDiagram participant names (`Loop`, `Alt`, `Note`, `Par`, `Opt` collide with control-block keywords → rename, e.g. `LoopSvc`).
     - Invalid arrow syntax (`-.x.->` is not a valid Mermaid arrow → use `--x` for solid-cross or `-.->` for dashed).
     - Literal `\n` in flowchart labels (use `<br/>` instead).
   - Fix surgically and re-validate. Repeat until 100% pass. Document the per-block fix in the commit message so future fan-out agents learn from it (Stage 8 pattern capture).
6. When BOTH schema-gate dry-run AND Mermaid batch validation pass (zero failed blocks), surface the consolidated branch to the operator for review.
7. Operator reviews; Architect addresses any final findings.

**MUST.**
- Schema gate passes (zero `[subsection-schema] FAIL` lines).
- **Mermaid batch validation passes (`mmdc` exits 0 for every block; zero `Parse error` messages).**
- License audit confirms zone declarations match `License_Boundary_Catalog.md`.
- Operator sign-off recorded before Stage 6.

**MUST NOT.**
- Run `/plan-to-project create --apply` until Stage 5 sign-off.
- **Skip Mermaid batch validation** — empirically, ~14% of agent-authored Mermaid diagrams have syntax errors that the skill's shallow P0-5 directive check (Stage 6 Phase 8) cannot detect (e.g. operator-observed Helm plan run 2026-04-29 caught 11/79 = 14% with parse errors only via batch `mmdc`). Skipping this validation means broken diagrams ship to the backlog and only surface during operator spot review (Stage 7) or downstream worker dispatch.

---

## Stage 6 — Backlog Seeding

**Purpose.** Materialize the plan as GitHub issues in the target Project V2 with proper field IDs, sub-issue relationships, blocking labels, and priorities.

**Process.**
1. Run preflight: `python3 -m scripts.create_issues preflight --org <ORG> --repo <REPO> --project <NUM>`
2. Run create: `python3 -m scripts.create_issues create --plan <PATH> --org <ORG> --repo <REPO> --project <NUM>`
3. Run sub-issue relationships: `python3 -m scripts.set_relationships --manifest manifest.json --repo <REPO>`
4. Run project field assignment: `python3 -m scripts.set_project_fields --manifest manifest.json --config manifest-config.json --org <ORG> --project <NUM>`
5. Run compliance check: `python3 -m scripts.compliance_check --manifest manifest.json --repo <REPO>`
6. Archive `manifest.json` + `manifest-config.json` for downstream scripts.

**MUST.**
- Preflight passes (Issue Type IDs + Project V2 field IDs present).
- Compliance check returns no P0 gaps (TDD language present, Security/Compliance present on mutation issues, dependency tables on blocked issues).

**MUST NOT.**
- Skip the manifest archival step (subsequent refresh / queue-order scripts depend on it).
- **MUST NOT pass `--allow-shallow-subsections`** to `plan-to-project create` / `refresh` / `amend`. See Self-Healing R-19. If the plan does not satisfy FR #45 schema, return to Stages 2–4 to deepen it before seeding. The 2026-05-09 #271 incident is the canonical RCA: bypass-flag-created subtrees produce `_TBD_` skeletons that are unworkable for Workers and one-way regress on subsequent refresh.

---

## Stage 7 — Operator Review + Refresh Loop

**Purpose.** Verify rendered issue bodies match operator quality bar; iterate via targeted refreshes if gaps surface.

**Process.**
1. Operator reviews the created issues in the GitHub Project.
2. Spot-check 5 issues per depth (Scope / Initiative / Epic / Story / Task) for: subsection completeness, content quality, license zone correctness, cross-issue references.
3. If gaps surface:
   - Re-author plan source for the affected items (back to Stage 4 if depth-fill is needed)
   - Run `python3 -m scripts.create_issues refresh --plan <PATH> --repo <REPO> --scope-issue <SCOPE> --apply` to regenerate bodies
   - Re-spot-check
4. If no gaps: mark the plan ready for orchestrator dispatch.

**MUST.**
- Spot-check covers all 5 hierarchy depths.
- Refresh runs idempotently (re-running with no plan changes produces zero updates).

---

## Stage 8 — Pattern Capture & Continuous Improvement

**Purpose.** Capture lessons from this plan for the next plan; evolve the skill + this document.

**Process.**
1. File issues for skill enhancements that surfaced (e.g. canonical full-depth template flag, parser fixes).
2. Document new patterns in `references/<pattern>.md` in the skill repo (e.g. `references/sub-agent-fan-out-pattern.md`).
3. Update root `CLAUDE.md` with any new gotchas.
4. Refine THIS document (`multi-sub-agent-plan-build-stages.md`) with edge cases, exemplars, anti-patterns surfaced during execution.

---

## Canonical Full-Depth Subsection Checklist

Every item (Scope / Initiative / Epic / Story / Task) MUST have these subsections by end of Stage 4:

| Subsection | Required at | Purpose |
|---|---|---|
| Heading (`#` to `#####`) | All | Canonical depth: Scope=`#`, Initiative=`##`, Epic=`###`, Story=`####`, Task=`#####` |
| User Story (As-I want-So that) | Story, Task | Fenced code block with concrete role / what / outcome |
| TL;DR | Story, Task | 1-2 sentence change summary |
| Why This Matters | All | 2-4 sentences grounding in concrete failure modes if NOT done |
| Assumptions | All | Roles / Starting point / Preconditions bullets |
| Dependencies | All | Table: Ticket \| Description \| Status |
| MoSCoW Classification | Story, Task; recommended for Epic | Must / Should / Could / Won't with concrete bullets; Won't-Have references sibling Stories |
| Success Criteria | Scope, Initiative, Epic | Testable bullets |
| Feature Scope | Scope, Initiative, Epic | Table or bulleted list of capability areas |
| I Know I Am Done When | All | 6-9 testable bullets; **last bullet MUST be TDD** ("failing test written BEFORE implementation") |
| Acceptance Criteria | Story, Task | 3-5 Given/When/Then scenarios with concrete values (real type names, real HTTP codes, real file paths) |
| Constraints | Story, Task | Bullets covering license zone, performance, security non-negotiables |
| Implementation Notes | Story, Task | Bulleted plan with concrete file paths + type names |
| Security/Compliance | All | Generic checklist (input validation / no secrets / least-privilege) PLUS per-feature security notes |
| Artifacts | All | Deliverables table — files, types, tests, docs, configs, Kafka topics produced/consumed |
| Mermaid Diagrams | All | ≥1 fit-to-purpose: sequence / state / flowchart / C4 (Context/Container/Component) |
| Code Areas to Examine | Epic | Table: Type \| Object \| Location \| Notes |
| Questions for Tech Lead | Epic | Bulleted refinement questions |

**FR #45 schema gate** (per `kdtix-open/skill-plan-to-project` plan-format.md) enforces a SUBSET of the above. The full-depth checklist is operator-stated quality bar — STRICTER than the schema gate. Pass the schema gate AND the full-depth checklist before Stage 6.

---

## Anti-patterns

- **Single-agent monolithic build.** Has been observed to time out at 17–36 minutes with zero issue writes for backlogs >50 issues. Always fan out per Epic.
- **Skipping Stage 0 recon.** Architect makes uninformed design decisions; license zone gets misclassified; cross-plan dependencies missed.
- **Skipping Stage 3 audit.** Stage-4 enrichment agents author redundantly or miss gaps; no per-item Mermaid coverage.
- **Skipping Stage 5 schema-gate dry-run.** `/plan-to-project create` fails at issue-creation time, partial backlog created, manual cleanup required.
- **Letting Stage-2 agents invent Stories.** Operator-stated story list MUST be passed to each agent. Free-form story authoring produces inconsistent Story granularity across the plan.
- **Touching Project Scope or Initiative content from Stage-2 agents.** Only the Architect modifies these levels. Stage-2 agents are scoped to ONE Epic.

---

## Cross-references

- **Skill canonical fan-out pattern doc:** `kdtix-open/skill-plan-to-project/references/sub-agent-fan-out-pattern.md`
- **Skill plan-format reference:** `kdtix-open/skill-plan-to-project/references/plan-format.md`
- **Project root CLAUDE.md** "Multi-sub-agent plan build" section
- **`docs/plans/Plan-of-Plans.md`** "Build process — MUST follow Stages 0–8" header
- **`.github/copilot-instructions.md`** Required-files list

---

_Ratified: 2026-04-29 | Owner: kdtix-open / Chris Kreager | First exemplar: Operations Helm Local-First 24×7 + Cloudflare Publish KDTIX-format plan (`plan/oh-helm-local-first-foundation` branch on `kdtix-open/agent-project-queue`)_
