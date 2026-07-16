---
title: "Everything fails silently"
tags:
  - meta
  - reliability
  - hub
date: 2026-07-16
---

> **In plain terms:** Almost every real problem in this garden has the
> same shape: something broke, and nothing anywhere looked broken. No
> crash, no error message, no red light — just a system quietly doing
> less than it claimed while every dashboard stayed green. This note is
> the map of that pattern: the incidents, why software fails this way,
> and the habit that catches it.

Loud failures are a gift. A crash points at itself: you get a stack
trace, a timestamp, an obvious place to start. The failures that cost
me real time never did that. They all passed every check that existed
— because the checks were measuring the wrong thing.

## The gallery

Every one of these was live for days or months before being noticed:

- **The deploys that stopped happening.** My auto-deploy watcher's
  Docker client was
  [[watchtower-push-to-live|too old for the server, so every poll errored before doing anything]].
  The container stayed "healthy." Builds stayed green. The site just
  quietly stopped updating.
- **The padlock that vanished mid-visit.** A one-line server default
  was [[nginx-https-downgrade-behind-cloudflare-tunnel|redirecting visitors from the secure site to the insecure one]].
  Browsers follow along without complaint; it only surfaced when the
  contact form died.
- **The pages that were 404s wearing a 200.** After a framework
  upgrade, [[nextjs-14-to-16-silent-bugs|every project page rendered "not found" while reporting success]]
  — one bug hiding a second bug behind it.
- **The menu link that did nothing.** A component quietly
  [[next-link-swallows-external-urls|swallowed clicks on an external URL]].
  No error. The click just didn't go anywhere.
- **The settings that were never applied.** Security rules sat in a
  config file that my setup
  [[config-that-silently-does-nothing|reads, accepts, and completely ignores]]
  — next to a manifest promising icons that never existed.

## Why software fails this way

Three reasons keep recurring. **Errors get absorbed:** somewhere
between the failure and you, a layer catches the problem and carries
on — the browser follows the insecure redirect, the poller retries
next minute, the missing icon just doesn't render. **Health checks
measure effort, not outcome:** "the container is running" and "the
build passed" are claims about the machinery, not about the result you
actually care about. **Absence doesn't alarm:** monitoring notices
events that happen. A deploy that *doesn't* happen, a header that
*isn't* sent, a file that *isn't* there — non-events trigger nothing.

## The habit that catches it

Don't ask the machinery whether it ran. Ask the outcome whether it
exists. Curl the live site and read the headers that actually came
back. Push a trivial visible change and watch it arrive. Request the
file the manifest promises. The strongest version is automated:
[[ci-that-checks-the-live-site|my CI now refuses to go green until the live site proves it's serving the new version]]
— and on its very first real run it caught ten days of deploys that
had silently gone nowhere.

## What I learned

Green is not evidence; it's the absence of evidence. Every incident in
this gallery survived because a check somewhere reported success about
the wrong thing. The fix is never more dashboards — it's one blunt
question aimed at the outcome itself: *show me, live, right now.*
This note grows as the pattern claims new victims. It will.

Related: [[ci-that-checks-the-live-site|A green build isn't a deploy]], [[watchtower-push-to-live|How my site updates itself minutes after I save my work]], [[delete-the-vaporware|Why I deleted my most impressive-looking case study]]
