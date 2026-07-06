---
title: "The HTTPS downgrade nginx hid behind Cloudflare Tunnel"
tags:
  - nginx
  - cloudflare
  - debugging
  - infra
date: 2026-07-06
---

My portfolio runs as a static export in an nginx container, published
through a Cloudflare Tunnel. Cloudflare terminates TLS and forwards to the
origin over **plain HTTP** — which nginx took personally.

## The symptom

A client-side POST to Web3Forms (the contact form backend) failed, but only
on some routes. Pages loaded fine over HTTPS. Nothing in the console
pointed at the server.

## The chain

nginx's automatic directory redirects (`/projects` → `/projects/`) build an
**absolute** `Location:` header using nginx's *own* scheme. Since the
tunnel talks to nginx over HTTP, every trailing-slash redirect said:

```
Location: http://ariefrse.my/projects/
```

The visitor arrived on HTTPS, hit a redirect, and got silently downgraded
to HTTP. Browsers follow it without complaint — the page still renders.
But now any client-side call to a CORS-restricted third party fails,
because the page origin is suddenly `http://`.

One line fixes it:

```nginx
absolute_redirect off;
```

Relative redirects (`Location: /projects/`) let the browser keep whatever
scheme it came in on.

## The bug behind the bug

The contact form had been "working" the whole time — because it faked
success with a `setTimeout` and never sent anything anywhere. Wiring it up
for real (FormData POST, not JSON, to avoid the CORS preflight) is what
exposed the downgrade. A fake implementation doesn't just fail to deliver;
it *masks* the infrastructure bugs a real one would have surfaced months
earlier.

## The lesson

When TLS terminates upstream of your origin, audit every place the origin
constructs absolute URLs — redirects, canonical links, sitemaps. `curl -sI`
against the origin and read the `Location:` headers; the browser hides the
downgrade from you.

Related: [[watchtower-push-to-live|Push-to-live with CI images + Watchtower]], [[nextjs-14-to-16-silent-bugs]]
