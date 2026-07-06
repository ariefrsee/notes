---
title: "Delete the vaporware case study"
tags:
  - portfolio
  - writing
  - evidence
date: 2026-07-06
---

My portfolio's AI page used to feature a "Maritime API Contract Inspector"
— a fully-designed five-stage pipeline with polished diagrams. It read
great. It was never built.

I deleted it. Polished presentation of something unverifiable doesn't add
credibility, it *spends* it: anyone who asks one follow-up question finds
the hole, and now everything else on the page is suspect too.

What replaced it is a data file of evidence entries sourced from actual
diffs and commit messages — the payroll parity bug an agent audit caught
(including the human revert of its one wrong fix), the HTTPS downgrade
root-caused on this very site, the authorization hardening pass before a
release. Each entry traces to a commit someone could go read.

Two things made this work:

- **Structure as a forcing function.** The evidence lives in a typed data
  array, not free-form page copy. Adding an entry means filling in fields
  like *what happened* and *where's the proof* — there's no slot for
  vibes.
- **Failures count as evidence.** The entry where the agent's fix was
  wrong and a human reverted it is *more* convincing than the successes,
  because it shows the review loop actually runs.

Same principle as this garden: everything traces back to something I
actually shipped or broke. A shorter list of true things beats a longer
list of plausible ones.

Related: [[gap-finder-audits]], [[spec-first-agentic-builds]]
