---
title: "A green build isn't a deploy: making CI check the live site"
tags:
  - infra
  - ci
  - deployment
date: 2026-07-16
---

> **In plain terms:** My website updates itself automatically after I
> save my work — but the "success" light only confirmed the package was
> *shipped*, not that it *arrived*. When the delivery robot on my
> server silently froze, I had days of green checkmarks and a stale
> website. Now the pipeline calls the live site and asks "which version
> are you running?" — and turns red if the answer is wrong.

My deploy pipeline looked solid: push to main, GitHub Actions builds a
Docker image, and [[watchtower-push-to-live|Watchtower on my server pulls it live within a minute]].
Every step reported success.

Then Watchtower froze — its
[[watchtower-push-to-live|ancient Docker client got rejected on every poll]]
— and the pipeline's blind spot became obvious. Builds kept passing.
Images kept pushing. The site kept serving a version from days ago.
Nothing anywhere was red, because every stage only vouched for
*itself*: the build vouched for the build, the push for the push, and
nobody's job was to confirm the site actually changed.

The fix has two small parts:

- **The site now carries its own version tag.** The build bakes the
  git commit ID into a `build-commit` meta tag in the page's HTML —
  invisible to visitors, but any script can read it and know exactly
  which commit production is serving.
- **CI's last job interrogates production.** After the image is
  pushed, a `verify-deploy` job polls the live site every 30 seconds
  for up to 10 minutes, comparing that tag to the commit it just
  built. Match → green. No match after 10 minutes → the workflow
  fails loudly, on the exact commit that didn't arrive.

A frozen Watchtower can still freeze — I can't reach into that box
from CI. But it can no longer freeze *silently*. The failure now shows
up minutes after the push, attached to the right commit, instead of
days later as a vague "why is the site old?"

## What I learned

"The pipeline succeeded" and "the deployment happened" are different
claims, and most pipelines only make the first one. Every stage
reporting success proves each stage *ran* — not that the end state you
care about exists. The cheapest insurance is to make the final stage
check the outcome directly: ask the live system what version it's
running and refuse to go green on the wrong answer. It's the automated
version of a lesson I'd already learned by hand — after wiring any
auto-deploy loop, verify the loop itself.

Related: [[watchtower-push-to-live|How my site updates itself minutes after I save my work]], [[known-ci-footguns|Teaching AI assistants to remember past mistakes]], [[nginx-https-downgrade-behind-cloudflare-tunnel|How my website quietly lost its security padlock]]
