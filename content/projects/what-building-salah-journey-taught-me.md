---
title: "What building a prayer app taught me about coding"
tags:
  - projects
  - learning
  - react
  - mobile
date: 2026-07-16
---

> **In plain terms:** I learn programming by building real things and
> shipping them, not by finishing tutorials. Building
> [[salah-journey|Salah Journey]] forced me through problems no course
> covers: syncing text to audio by hand, turning a website into phone
> apps, and cutting features I liked so the thing could ship. Each
> problem taught me more than a month of lessons would have, because I
> couldn't skip to the answer.

Tutorials have a quiet flaw: someone already removed all the real
problems. The code works, the data exists, the scope is fixed. Building
[[salah-journey|Salah Journey]] — a real app with real users — put the
problems back in. Three of them taught me the most.

**The data nobody hands you.** The app's core feature is
karaoke-style synced text: each line of a recitation highlights in
time with the audio. Every music app has this, so I assumed it was a
solved problem. It is — *if* you have the timing data. I didn't.
Every line of every recitation needed a start and end time, and the
only source was me, listening and noting timestamps. Doing that by
hand in a text file was miserable, so I ended up building a tool
inside the app itself: a **timing calibrator** with an audio timeline,
playback controls, and a line editor — press a key when a line starts,
press it when it ends, export the data. Building a tool to build the
thing felt like a detour. It wasn't. The calibrator turned hours of
error-prone squinting into minutes, and taught me more about audio
APIs and state management than the feature it supported.

**The gap between "runs on the web" and "works on a phone".** Capacitor
promises you wrap your web app and get Android and iOS for free. The
wrapping part is true. But the qibla finder needs the compass and GPS,
prayer reminders need notification permissions, and each platform has
its own rules about asking. "One codebase, three platforms" is real —
the free part isn't. Learning where the web abstraction ends and the
native platform begins is exactly the kind of knowledge a tutorial
can't give you, because it only shows up when a real phone says no.

**Shipping means deleting.** The app once had Malay translations
alongside English, and more screens than it has now. Every extra
surface was one more thing to keep correct in an app where correctness
is the whole point — you don't ship a *roughly right* prayer guide. I
cut the translations and consolidated the prayer flow into one shared
component. The app got smaller and better at the same time, which
still feels backwards and no longer surprises me. (The same instinct
later led me to [[delete-the-vaporware|delete my most impressive-looking case study]].)

## What I learned

The pattern behind all three: **real projects don't let you skip the
hard part**, and the hard part is where the learning lives. A tutorial
would have handed me the timing data, mocked the compass, and fixed
the scope. Because nothing was handed to me, I now actually know how
to sync audio to text, where web-to-native wrappers leak, and how to
cut scope without cutting quality. If you want to learn to code, pick
something you genuinely want to exist and build it until strangers can
use it. The gaps you fall into on the way — those are the curriculum.

Related: [[salah-journey|An app that teaches you to pray, starting from zero]], [[spec-first-agentic-builds|Write the plan before the code]], [[delete-the-vaporware|Why I deleted my most impressive-looking case study]]
