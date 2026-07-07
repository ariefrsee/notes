---
title: "Upgrading a website's engine: the bugs that don't crash"
tags:
  - nextjs
  - migration
  - debugging
date: 2026-07-03
---

> **In plain terms:** Websites are built on frameworks that get major
> upgrades, like swapping a car's engine. Some problems after the swap are
> loud — the car won't start, you fix it immediately. The dangerous ones
> are quiet: everything looks fine, but a page that should show a project
> now politely says "not found." On my own site, that quiet failure ran
> for months before I caught it.

The compile errors in a Next.js 14 → 15/16 migration (Next.js is the
framework this site and many others are built on) fix themselves — the
build tells you. The dangerous changes are the ones that keep rendering:

- **Dynamic route params became Promises.** In 14, `params` is a plain
  object; from 15 on it's a `Promise`. Code reading `params.slug` without
  `await` doesn't throw — it reads a property off a Promise, gets
  `undefined`, and the page quietly renders its empty or not-found state.
  Same for `searchParams`.
- **`fetch` caching defaults flipped** in 15: pages silently change between
  cached and per-request behavior with zero code changes.
- **`cookies()` / `headers()` went async** — sync usage survives loose type
  checking and only fails on the runtime paths that touch it.

The canonical fix:

```tsx
- export default function Page({ params }: { params: { slug: string } }) {
-   const item = getBySlug(params.slug)
+ export default async function Page({ params }: { params: Promise<{ slug: string }> }) {
+   const { slug } = await params
+   const item = getBySlug(slug)
```

## The war story

I found this class of bug live on my own portfolio: **every project detail
page had been rendering a 404 for months while returning HTTP 200.** The
`[slug]` page read `params.slug` synchronously → `undefined` →
`notFound()` — but the server kept serving a prerendered shell, the listing
page worked, and nothing ever logged an error.

Fixing it unmasked a second buried bug: the page was a server component
rendering framer-motion components directly, which errors at build time —
`notFound()` had been bailing out before that code path ever executed, so
the build always "passed."

## What I learned

On this bug class, *"it builds and the page loads"* is exactly what failure
looks like. After a migration, drive every dynamic route in a real browser
and confirm actual data renders. A `curl` returning 200 proves nothing —
the 404 happens after hydration.

Related: [[known-ci-footguns|Teaching AI assistants to remember past mistakes]], [[gap-finder-audits|Letting an AI hunt for the bugs humans skim past]]
