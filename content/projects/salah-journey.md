---
title: "An app that teaches you to pray, starting from zero"
tags:
  - projects
  - react
  - mobile
date: 2026-07-16
---

> **In plain terms:** I built and shipped a free app called Salah
> Journey for people who want to learn the Muslim daily prayers but
> don't know where to start. It shows the Arabic words, how to
> pronounce them, and what they mean — all highlighted in sync with
> audio, like karaoke lyrics. No account, no ads, and it runs in the
> browser or as a phone app. It's live at
> [salah-guide.netlify.app](https://salah-guide.netlify.app/).

Learning to pray as an adult is intimidating. The resources that exist
mostly assume you already know the basics: they give you the Arabic
text and expect you to figure out the rest. If you're starting from
zero — a revert, someone returning to prayer, a parent teaching a kid
— you need something that walks you through *every* step without
judgment.

That's the gap [Salah Journey](https://salah-guide.netlify.app/)
fills. The core idea is borrowed from music apps: **synchronized
lyrics**. Every recitation in the prayer — from the opening takbir to
the closing salam — plays as audio while the Arabic text, a
transliteration you can actually pronounce, and the English meaning
highlight line by line in time with the recording. You can slow the
audio down and loop a single line until it sticks, recite along with
the text hidden, or test yourself from memory.

Around that core sits everything else you need in one place: a
step-by-step wudu (purification) guide, location-based prayer times, a
compass qibla finder that points toward Makkah, a tracker for missed
prayers you're repaying, a tasbih counter, and a mosque finder on a
live map.

Technically it's one React + Vite codebase (with shadcn/ui) that ships
three ways: as a website on Netlify, and as Android and iOS apps via
Capacitor. Same code, three platforms.

Deliberate choices: free, no login, no ads. An app meant to remove
barriers to worship shouldn't add a signup wall in front of them.

## What I learned

The feature that makes the app work — synced lyrics — is also the one
that nearly didn't ship, because someone has to produce the timing
data for every line of every recitation. The story of how I solved
that (and everything else this project taught me) is its own note:
[[what-building-salah-journey-taught-me|What building a prayer app taught me about coding]].

Related: [[what-building-salah-journey-taught-me|What building a prayer app taught me about coding]], [[delete-the-vaporware|Why I deleted my most impressive-looking case study]]
