---
title: "Letting an AI hunt for the bugs humans skim past"
tags:
  - agentic-engineering
  - auditing
  - human-in-the-loop
date: 2026-07-03
---

> **In plain terms:** Many software systems have two doors into the same
> data — say, a screen employees use and a back-office screen admins use.
> Over time the two doors quietly stop behaving the same, and nobody
> notices because checking is tedious. I give that tedious checking job to
> an AI assistant, which found real problems — including one where the
> same employee's salary came out different depending on which door you
> used.

A gap-finder audit is a focused agent task: *compare every code path in
surface A against its equivalent in surface B and report behavioral gaps.*
It works because parity bugs are exactly what humans skim past and agents
grind through.

On my HRMS (a human-resources management system), the audit brief was
"compare every API code path against its Filament admin-panel equivalent"
— that is, the programmatic interface versus the admin's point-and-click
screens. Real findings that shipped as fixes:

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

## What I learned

Agents and humans are good at opposite halves of auditing: the agent
never gets bored grinding through hundreds of code-path comparisons, and
the human knows which of its findings — and which of its *fixes* — the
business can actually live with. The payroll bug proved the first half;
the same-day revert proved the second. I'd trust neither alone.

Related: [[hrma-built-by-ai-agents|hrma: an HR system built almost entirely by AI agents]], [[known-ci-footguns|Teaching AI assistants to remember past mistakes]], [[spec-first-agentic-builds|Write the plan before the code]], [[authz-hardening-sweep|The sweep that found managers could pay other teams' claims]]
