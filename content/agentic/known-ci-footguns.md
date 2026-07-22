---
title: Teaching AI assistants to remember past mistakes
tags:
  - agentic-engineering
  - context-engineering
  - ci
date: 2026-07-03
---

> **In plain terms:** AI coding assistants start every work session with a
> blank memory, so they can repeat a mistake that was already caught and
> fixed last month. My fix: keep a short "never do this again" list in a
> file the assistant is required to read before touching any code. Every
> mistake a human reviewer catches gets added to the list, so it can only
> happen once.

Agents forget between sessions; the repo doesn't.

On my HRMS project, a seeder bug (duplicate employee IDs from an
un-deduplicated `pluck`) silently broke CI — the automated pipeline that
tests and ships every change. The fix was one line. The durable fix was a
standing section in the repo's `CLAUDE.md` that every agent session loads
before touching code:

```markdown
## Known CI Footguns — Never Re-introduce
1. Seeder dedup: always ->unique() on pluck('employeeId') in
   MockDataSeeder — duplicate IDs will break seeding silently
2. Leave calendar scoping: non-approver roles must only see
   their own leaves — global scoping breaks this
3. Staging Docker BuildKit cache is flaky — pre-existing,
   unrelated to code changes
```

The rule: every regression a reviewer catches becomes a constraint the next
agent session *starts with*, instead of a mistake it gets to repeat.

Why this works better than a wiki:

- The context file lives next to the code it protects, so it can't drift out
  of the workflow — agents read it whether they "remember" to or not
- Each entry is phrased as a constraint ("never re-introduce"), not a
  post-mortem — agents act on imperatives better than narratives
- It stays short. If it grows past a screen, entries get promoted into lint
  rules or tests, which are even harder to ignore

## What I learned

Don't fight an AI assistant's forgetfulness — design around it. The
repo remembers so the agent doesn't have to, and a mistake written down
as a constraint ("never re-introduce X") only ever happens once. The
discipline that makes it work is keeping the list short: once an entry
matters enough, it graduates into a test or a lint rule, which even a
forgetful agent can't walk past.

Related: [[hrma-built-by-ai-agents|hrma: an HR system built almost entirely by AI agents]], [[gap-finder-audits|Letting an AI hunt for the bugs humans skim past]], [[parallel-agent-sessions|Running several AI assistants at once]]
