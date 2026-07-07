---
title: Running several AI assistants at once (and staying in charge)
tags:
  - agentic-engineering
  - workflow
date: 2026-07-03
---

> **In plain terms:** Instead of working with one AI assistant and waiting
> while it finishes each task, I run several at the same time — one
> writing code, one testing, one writing documentation — the way a manager
> runs a small team. My job shifts to assigning the work, checking it, and
> making the final calls.

I run Claude Code and OpenAI Codex as parallel sub-agents: implementation,
debugging, and documentation happening concurrently instead of serially.
This note is the operating model, condensed.

## The loop

1. **Context & plan** — give each agent clear goals, relevant context,
   constraints, API assumptions, and an explicit completion contract
2. **Orchestrate & build** — break work into bounded tasks, connect the
   right tools, coordinate implementation across the stack
3. **Verify & debug** — inspect outputs, compare payloads, run checks,
   trace failures, iterate until acceptance criteria pass
4. **Human review** — stay at every consequential decision; document
   tests, tradeoffs, and handover notes

## What parallelism is actually for

Not typing speed — *waiting*. While one session runs a build or a test
suite, another is drafting the migration, a third is writing the PR
description. The human becomes the scheduler and reviewer, which is the
job that was always worth the salary.

## Boundaries that keep it sane

- One session per bounded task, not one session per project — sessions
  that sprawl accumulate stale context and drift
- Durable knowledge goes in the repo ([[known-ci-footguns|CLAUDE.md]],
  specs, docs) — never rely on a session "remembering"
- Every merge is human-gated. The commit trail carries the AI co-author
  trailers, so the split of labor is auditable, not vibes

## What I learned

The productivity isn't in the agents typing faster — it's in never
sitting idle while something runs. My job stopped being "write the
code" and became "keep three streams of work unblocked and review what
comes back," which is a management skill, not a typing skill. The
surprise: the boundaries (one session per task, knowledge in the repo,
human-gated merges) matter more than the number of agents.

Related: [[spec-first-agentic-builds|Write the plan before the code]], [[gap-finder-audits|Letting an AI hunt for the bugs humans skim past]]
