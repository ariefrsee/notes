---
title: "The menu link that did nothing when clicked"
tags:
  - nextjs
  - debugging
  - app-router
date: 2026-07-06
---

> **In plain terms:** My website's menu had a button linking to this very
> notes site. Clicking it did… nothing. No error, no warning — the button
> even lit up as if it had worked. The cause: the site's internal
> "navigation system" was asked to go to an address outside the site,
> which it can't do, and it failed without telling anyone. The fix was to
> mark outside links as ordinary links so the browser handles them itself.

Another entry in the [[nextjs-14-to-16-silent-bugs|bugs that don't crash]]
family: in the App Router, wrapping a **cross-origin URL** in `next/link`
can fail without a trace. The click is intercepted, the client router tries
to resolve a route it doesn't own, and then… nothing. No navigation, no
console error, no 404. On my portfolio the content area went blank and the
nav item lit up as active — the UI *looked* like it worked.

## How it got in

The sidebar renders nav items from a config array. Adding an external item
felt free:

```ts
{ id: 'notes', label: 'Notes', href: 'https://ariefrsee.github.io/notes/' }
```

Every item went through the same `<Link href={item.href}>` — internal
routes worked, so the pattern looked proven. Nobody clicked the one
external item before shipping, and there was nothing to catch: the build
passes, the link renders, `curl` sees a normal `<a href>` in the HTML. The
failure only exists in the click handler after hydration.

## The fix

Branch on the href shape and give external links a plain anchor:

```tsx
export function isExternalHref(href: string): boolean {
  return /^https?:\/\//.test(href)
}
```

```tsx
{isExternalHref(item.href) ? (
  <a href={item.href} target="_blank" rel="noopener noreferrer">…</a>
) : (
  <Link href={item.href}>…</Link>
)}
```

Two follow-on details worth encoding:

- **Mobile drawers close on route change.** An external link opening a new
  tab never changes `pathname`, so the `useEffect` that closes the drawer
  never fires — wire `onClick={onClose}` on the anchor explicitly.
- **Mark it visually.** An outbound-arrow icon replaces the badge slot, so
  users know they're leaving the site.

## The lesson

A shared "render everything through `<Link>`" abstraction is a trap the
moment one item is off-origin. Same rule as the migration bugs: click every
nav item in a real browser — the ones that fail silently are exactly the
ones a build can't catch.

Related: [[nextjs-14-to-16-silent-bugs|Upgrading a website's engine: the bugs that don't crash]], [[known-ci-footguns|Teaching AI assistants to remember past mistakes]]
