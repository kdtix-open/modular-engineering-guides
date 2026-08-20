# Intent vs Implemented

> **STATUS: STAGED DRAFT (2026-08-19) — not Core Principle #7.**
> Record Intent as a first-class artifact. Implementation must remain a
> complement of current Intent. An adversary change is illegal until Intent
> is amended on an auditable ledger — **after Operator ratification**.

Cross-reference (APQ overlay): `.github/docs/philosophy/PURPOSE.md` is that
repo's mission ledger, not a substitute for this principle.

---

## What Intent is

**Intent** is the stated *why and must-remain-true* at the time of first
implementation. It lives in more than one place; all of them count:

- failing tests and test names (executable Intent)
- public-API docs / JSDoc that state validity conditions
- issue and plan headings plus acceptance criteria
- architecture notes, ADRs, and mermaid diagrams

**Implemented** is the code, config, and changing dependencies that exist now.

## Complement vs adversary

A later change is a **complement** when it still serves current Intent
(peer feature, compatible extension).

A later change is an **adversary** when it cancels Intent, creates a
dead-end, or loops with an older rule. Adversary work must **stop** until
the Intent ledger is updated (old Intent, new Intent, reason, artifacts
touched). Updating tests to paper over broken code is adversary drift.

## Change-order ledger (minimum fields)

| Field | Required |
|---|---|
| Date (UTC) | yes |
| Old Intent (quote or link) | yes |
| New Intent | yes |
| Reason (change order / new architecture / dependency) | yes |
| Artifacts updated (tests first, then code, then headings/docs) | yes |

Prefer in-repo records (issue body, ADR, test, this guide's project overlay).
Do not rely on chat history.

## I know I am done when

- A reviewer can find the original Intent without archaeology
- Complement work cites current Intent
- Adversary work has a ledger entry before merge
