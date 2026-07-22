---
title: "The sweep that found managers could pay other teams' claims"
tags:
  - agentic-engineering
  - security
  - human-in-the-loop
date: 2026-07-22
---

> **In plain terms:** Before letting real employees use my HR system, I ran
> an AI-driven security sweep asking one question everywhere: not "does
> this feature work," but "is this person actually allowed to do this." It
> found a manager could approve a claim for someone outside their own
> team, every employee could see the whole company's approved leave
> instead of just their own, and the people allowed to approve payments
> were hardcoded into the code instead of configurable. All fixed before
> anyone touched the live system.

A [[gap-finder-audits|gap-finder audit]] asks "does this behave the same
everywhere." An authorization sweep asks a narrower, sharper question:
*who is allowed to do this, and does the code actually check?* I ran one
across my HRMS's claims, payroll, and leave-calendar code before the pilot
release, specifically hunting for scope bugs rather than behavior bugs.

## What it found

- **Cross-team approval**: a manager could mark another team's claim as
  paid — the endpoint checked "are you a manager," not "is this your
  team"
- **Whole-company leave visibility**: every employee could see everyone's
  approved leave, not just their own team's, because the leave calendar
  had no per-role scoping at all
- **Hardcoded approvers**: the two employee IDs allowed to approve claims
  were baked into the code, so the approval chain would break the moment
  either of those two people left or changed roles
- **A role with the wrong panel access**: managers had access to the full
  admin panel instead of the manager-scoped one they actually use

## Why this is a different sweep than a gap-finder audit

Gap-finder audits compare two surfaces — API vs admin panel — and ask if
they agree. This sweep only has one surface; it asks whether the *access
control* around that surface matches who should actually be allowed to
act. Both are exhaustive-comparison tasks agents are well-suited to, but
they check for different classes of bug, and neither substitutes for the
other. I now run both before every release, not one instead of the other.

## What I learned

The scariest bugs in this sweep weren't crashes or wrong numbers — they
were correct-looking actions performed by the wrong person. A manager
successfully marking a claim as paid looks identical whether it's their
team's claim or not; nothing anywhere flags it. That's the same shape as
[[everything-fails-silently|everything else that costs me real time]]:
the system did exactly what the code said, and the code just didn't say
enough. Fixed before the pilot's first real payroll run — the version of
"before" that actually matters.

Related: [[gap-finder-audits|Letting an AI hunt for the bugs humans skim past]], [[everything-fails-silently|Everything fails silently]]
