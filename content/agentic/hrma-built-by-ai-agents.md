---
title: "hrma: an HR system built almost entirely by AI agents"
tags:
  - agentic-engineering
  - hrms
  - overview
date: 2026-07-22
---

> **In plain terms:** hrma is a real HR system — payroll, leave,
> attendance — used by a real company, and roughly seven out of every ten
> commits that built it were written by an AI agent, with a human
> reviewing and gating every one before it merged. This note is the
> starting point for everything else in this garden about how that
> actually works day to day.

## What hrma is

A PDPA-aware ([Malaysia's data protection law](https://en.wikipedia.org/wiki/Personal_Data_Protection_Act_2010)) HR
platform for Malaysian SMEs, packaged as single-tenant SaaS — one
deployment per customer company. The stack: Laravel 12 + Filament for the
HR admin panel, a React employee web app, a Capacitor-wrapped mobile app
for iOS and Android, and Postgres underneath. It is not a demo — a real
company pilots it for real payroll, leave, and attendance, hosted the way
[[hosting-an-hrms-on-an-always-on-pc|described here]].

## The build, in numbers

145 commits from first scaffold to today, spanning roughly nine weeks.
104 of them — about seven in ten — carry an AI agent's co-author trailer
in the commit message: implementation, fixes, deployment config, and
docs, not just boilerplate. Every one of those still went through the
same gate a human-written commit would: review before merge.

The repo also carries design documents for individual features —
role permissions, attendance geofencing, multi-company support — written
before the corresponding code, the same discipline described in
[[spec-first-agentic-builds|write the plan before the code]], just applied
project-wide rather than to one module map.

## What "AI agent built" actually looks like day to day

The number alone (seven in ten commits) undersells the interesting part,
which is *where humans stayed load-bearing*:

- **Review**: a [[gap-finder-audits|gap-finder audit]] compared the API
  against the admin panel and found a real payroll parity bug — and a
  human reverted the one agent-proposed fix that was technically correct
  and organizationally wrong.
- **Hardening**: an [[authz-hardening-sweep|authorization sweep]] caught
  managers who could approve claims outside their own team, before any
  real money moved.
- **Maintenance**: when a seeder bug broke CI silently, the fix became a
  [[known-ci-footguns|standing entry in the repo's own agent context]] so
  the next session can't reintroduce it.

None of these were the agent working alone, and none were a human
reviewing line-by-line either. It was closer to a hand-off: the agent
does the exhaustive grinding work — every route, every code path, every
CI log — and the human makes the one call at the end that the exhaustive
pass can't: *does this decision match how the business actually works.*

## What I learned

"Built end-to-end with AI agents" is true and also the less interesting
half of the story. The more useful fact is the shape of what stayed
human: not typing, but judgment — which fix to keep, which access to
grant, which failure is allowed to happen twice. Seven in ten commits
came from an agent; zero in ten decisions about what "correct" means for
this business did.

Related: [[gap-finder-audits|Letting an AI hunt for the bugs humans skim past]], [[authz-hardening-sweep|The sweep that found managers could pay other teams' claims]], [[known-ci-footguns|Teaching AI assistants to remember past mistakes]], [[hosting-an-hrms-on-an-always-on-pc|Hosting a company's HR system on a PC that never turns off]]
