---
title: Spec-first agentic builds
tags:
  - agentic-engineering
  - planning
  - obsidian
date: 2026-07-03
---

Docs before code. When I built CrewSync (a multi-tenant maritime crew
management SaaS), the first artifact wasn't a scaffold — it was an Obsidian
vault: architecture, tech stack, and one note per module (auth,
multi-tenancy, crew management, certificates, safe-manning compliance,
rotations, sign-on/sign-off, sea service records).

That vault became the agent context. The MVP foundation landed the next
day: 196 files, ~24k lines — against a written contract instead of
improvised scope.

## Why this works with agents specifically

- **Agents fill ambiguity with plausible inventions.** A written module map
  turns "build crew management" from an open-ended prompt into a bounded
  task with named entities and relationships.
- **The spec survives the session.** Context windows end; the vault
  doesn't. Session N+1 starts from the same source of truth as session 1.
- **Scope creep becomes visible as a diff.** If the agent builds something
  not in the vault, that's a flag — either the vault was wrong (update it)
  or the agent drifted (revert it).

## The discipline

1. One note per module, written in plain language, before any code
2. The notes state what's *out* of scope as explicitly as what's in
3. When implementation teaches you something, the note gets updated in the
   same session — the vault is living, not archival

This is the same reason [[known-ci-footguns|CI footguns live in CLAUDE.md]]:
durable context beats repeated explanation.

Related: [[parallel-agent-sessions]], [[gap-finder-audits]]
