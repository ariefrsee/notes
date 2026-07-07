---
title: "Hosting a company's HR system on a Mac"
tags:
  - infra
  - cloudflare
  - docker
  - self-hosting
date: 2026-07-07
---

> **In plain terms:** hr.ariefrse.my is a real HR system — payroll,
> leave, attendance — being piloted by a real company. It doesn't run in
> a data center. It runs in Docker on a Mac, and a free Cloudflare
> service carries traffic from the internet to that Mac through an
> outbound connection, so the machine never has to open a single door to
> the outside world. Monthly hosting cost: zero.

The pilot stack for hrmydeck (the HRMS built end-to-end with AI agents —
see [[known-ci-footguns|how its mistakes become agent context]]) is
three pieces, none of them a rented server:

- **The Mac** runs everything in Docker Compose: Postgres 16 and one
  Laravel container that serves all three surfaces — `/admin` (the
  Filament HR panel), `/employee` (the React staff self-service app),
  and `/api` (the mobile app's backend).
- **Cloudflare Tunnel** exposes it. `cloudflared` on the Mac dials *out*
  to Cloudflare and keeps the connection open; visitor traffic to
  `hr.ariefrse.my` rides back down it. No port forwarding, no public IP,
  no firewall holes — the Mac is unreachable except through the tunnel.
- **Hostinger holds the DNS**: a CNAME points `hr` at
  `<tunnel-uuid>.cfargotunnel.com`. Cloudflare terminates TLS, so the
  padlock works without managing certificates on the Mac.

The container boots self-sufficient: database migrations run on start,
the first boot seeds demo accounts, and secrets (app key, a separate
encryption master key for employee data) live in one `.env.staging`
file. Daily startup is two commands — `docker compose up -d` and
`cloudflared tunnel run`.

## The footguns

- **Don't run `cloudflared` inside Docker** pointing at `localhost` —
  that's the *container's* localhost, and every request 502s. The tunnel
  runs natively on the Mac; the app runs in Docker.
- **The app must know its public URL.** Laravel's session cookies are
  scoped to `STAGING_PUBLIC_URL`; if it doesn't match the browser's URL
  exactly, logins die with "Page expired" and nothing hints why.
- **Quick tunnels are for demos only.** `trycloudflare.com` URLs change
  on every restart; a named tunnel plus the DNS record is what makes the
  address permanent.

## The lesson

A production *pilot* doesn't need production *infrastructure*. The
architecture is honest about its stage: one machine, one compose file,
zero hosting bill — but with real TLS, a hidden origin, encrypted
employee data, and boot-time migrations, so graduating to a VPS later is
a `docker compose up` on different hardware, not a redesign.

Same tunnel trick as the portfolio — and the same class of surprise:
see [[nginx-https-downgrade-behind-cloudflare-tunnel|How my website quietly lost its security padlock]].

Related: [[watchtower-push-to-live|How my site updates itself minutes after I save my work]], [[spec-first-agentic-builds|Write the plan before the code]]
