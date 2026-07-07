---
title: Write the plan before the code — especially with AI
tags:
  - agentic-engineering
  - planning
  - obsidian
date: 2026-07-03
---

> **In plain terms:** AI assistants are eager — ask for something vague
> and they'll confidently build their best guess, which is often not what
> you meant. So before any code gets written, I write a plain-language
> plan describing every part of the product and what's deliberately left
> out. The AI builds against that plan, and anything it builds that isn't
> in the plan is treated as a red flag.

Docs before code. When I built CrewSync (a multi-tenant maritime crew
management SaaS), the first artifact wasn't a scaffold — it was an Obsidian
vault (a folder of linked notes): architecture, tech stack, and one note
per module (auth,
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

## What I learned

Writing the plan first felt slower and was faster: one day of writing
notes bought a 196-file MVP the next day, because the agent spent zero
time guessing what I meant. The deeper lesson is that a written spec
turns "the AI went off the rails" from a feeling into a diff — anything
built that isn't in the plan is visibly either the plan's fault or the
agent's, and both are fixable.

Related: [[parallel-agent-sessions|Running several AI assistants at once]], [[gap-finder-audits|Letting an AI hunt for the bugs humans skim past]]
