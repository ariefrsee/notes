---
title: "Gap-finder audits: letting an agent hunt parity bugs"
tags:
  - agentic-engineering
  - auditing
  - human-in-the-loop
date: 2026-07-03
---

A gap-finder audit is a focused agent task: *compare every code path in
surface A against its equivalent in surface B and report behavioral gaps.*
It works because parity bugs are exactly what humans skim past and agents
grind through.

On my HRMS, the audit brief was "compare every API code path against its
Filament admin-panel equivalent." Real findings that shipped as fixes:

- **Payroll parity**: the API's payroll path was missing claims
  reimbursement, unpaid-leave deductions, and custom deductions that the
  admin panel applied — two code paths computing *different salaries for
  the same employee*
- Onboarding via API never sent the welcome invite the admin flow sent —
  API-created employees silently never got credentials
- Exposed-but-unimplemented routes on a resource controller

## The part that matters: the revert

One agent fix was technically sound and organizationally wrong. Facing an
approval deadlock (no approvers configured), it fell back to "any
manager-role user can approve." No crash, tests pass — but approvers must
be *explicitly configured*, not inferred from a role. Human review caught
it; it was reverted the same day.

That's the point of human-in-the-loop: **agents optimize for "no
deadlock"; humans know which deadlocks are load-bearing.**

## Writing a good audit brief

- Name the two surfaces precisely (API vs admin panel, web vs mobile)
- Ask for *behavioral* gaps, not style differences
- Require each finding to state: the gap, the user-visible consequence,
  and the proposed fix — so review is triage, not re-derivation

Related: [[known-ci-footguns]], [[spec-first-agentic-builds]]
