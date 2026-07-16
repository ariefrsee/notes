---
title: "The settings that did nothing and the icons that didn't exist"
tags:
  - nextjs
  - nginx
  - pwa
date: 2026-07-16
---

> **In plain terms:** I found two kinds of quiet lies on my own
> website. First, a settings file full of security rules that were
> never actually applied — the tool reads them, says nothing, and
> ignores them in my setup. Second, an app manifest that promised
> icons and screenshots that had never existed; every device that
> asked for them got a "file not found". Nothing crashed either way.
> The site just silently wasn't what its own files claimed.

An audit of my portfolio turned up two failures with the same shape:
**declared but not delivered**. Neither threw an error. Both had been
live for months.

## The security headers that were never sent

My `next.config.ts` had a proper `headers()` block — content security
policy, frame protection, the lot. But this site is a static export
(`output: 'export'`): Next.js builds plain HTML files and my own nginx
serves them. In that mode, `headers()` and `redirects()` do nothing —
there's no Next server at runtime to apply them. Next.js doesn't warn
you; the config parses fine and is silently ignored.

So the rules moved to the thing that actually serves the pages:
`nginx.conf`. Which has its own trap waiting — nginx **drops all
inherited `add_header` directives** the moment a `location` block adds
any header of its own. Declare headers only at the top level and some
routes quietly lose every one of them. The fix is repeating the full
header set in each location that sets any header, verified with
`nginx -t` and then by actually curling the live responses.

## The manifest that promised phantom files

The PWA manifest — the file that tells phones how to install the site
as an app — referenced eight icon sizes, screenshots, and shortcut
icons. None of those files existed. The layout also linked
`/icon.svg` and `/apple-touch-icon.png`; also missing. Every browser
and device that trusted the manifest got 404s, invisibly, because
nothing on the page itself depends on them. The template had written
checks the repo never cashed.

The fix was to make the promises true *and* smaller: draw a real
`icon.svg` brand mark, generate every referenced PNG from it, and
rewrite the manifest to list only assets that exist.

## What I learned

Configuration is a list of claims, and nothing checks claims by
default. A settings block can parse cleanly and do nothing; a manifest
can reference files that were never created; in both cases the system
looks healthy because the failure lives in what's *absent* from
responses, not in any error. The audit that catches this is dumb and
effective: for every promise in config — a header, a file, a redirect
— go ask the live site for it and look at what actually comes back.
It's the same lesson as [[ci-that-checks-the-live-site|making CI check the live site]]:
trust what the running system serves, not what the files say.

Related: [[ci-that-checks-the-live-site|A green build isn't a deploy]], [[nginx-https-downgrade-behind-cloudflare-tunnel|How my website quietly lost its security padlock]], [[nextjs-14-to-16-silent-bugs|Upgrading a website's engine: the bugs that don't crash]]
