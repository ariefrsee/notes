---
title: "Hosting a company's HR system on a PC that never turns off"
tags:
  - infra
  - cloudflare
  - docker
  - self-hosting
date: 2026-07-07
---

> **In plain terms:** hr.ariefrse.my is a real HR system — payroll,
> leave, attendance — being piloted by a real company. It doesn't run in
> a data center. It runs in Docker on an always-on Windows PC, and a
> free Cloudflare service carries traffic from the internet to that
> machine through an outbound connection, so it never has to open a
> single door to the outside world. Monthly hosting cost: zero.

The pilot stack for hrmydeck (the HRMS built end-to-end with AI agents —
see [[known-ci-footguns|how its mistakes become agent context]]) is
three pieces, none of them a rented server:

- **The PC** runs everything in Docker Compose: Postgres 16 and one
  Laravel container that serves all three surfaces — `/admin` (the
  Filament HR panel), `/employee` (the React staff self-service app),
  and `/api` (the mobile app's backend).
- **Cloudflare Tunnel** exposes it. `cloudflared` on the machine dials
  *out* to Cloudflare and keeps the connection open; visitor traffic to
  `hr.ariefrse.my` rides back down it. No port forwarding, no public IP,
  no firewall holes — the PC is unreachable except through the tunnel.
- **Hostinger holds the DNS**: a CNAME points `hr` at
  `<tunnel-uuid>.cfargotunnel.com`. Cloudflare terminates TLS, so the
  padlock works without managing certificates on the machine.

The container boots self-sufficient: database migrations run on start,
the first boot seeds demo accounts, and secrets (app key, a separate
encryption master key for employee data) live in one `.env.staging`
file. The machine runs unattended: `cloudflared` is installed as a
Windows service that restarts if it dies, the containers carry
`restart: unless-stopped` so they return whenever Docker does, and a
scheduled task dumps the database nightly to a second location.

## The footguns

- **Don't run `cloudflared` inside Docker** pointing at `localhost` —
  that's the *container's* localhost, and every request 502s. The tunnel
  runs natively on the host; the app runs in Docker.
- **The app must know its public URL.** Laravel's session cookies are
  scoped to `STAGING_PUBLIC_URL`; if it doesn't match the browser's URL
  exactly, logins die with "Page expired" and nothing hints why.
- **Quick tunnels are for demos only.** `trycloudflare.com` URLs change
  on every restart; a named tunnel plus the DNS record is what makes the
  address permanent.

## The trade-off, honestly

What this buys:

- **RM0/month**, no certificates to renew, no server to rent
- **A hidden origin** — the machine dials out and never opens a port,
  which beats a carelessly firewalled VPS, not just matches it
- **Real hardware** — a desktop PC outruns a cheap VPS, and debugging
  production means sitting down at the machine, not SSHing somewhere
- **A clean exit** — compose file + migrations-on-boot + one `.env`
  means moving to a VPS later is a redeploy, not a redesign

What it costs:

- **Uptime rides on one machine and one internet connection.** The PC
  never turns off, but Windows updates reboot it on their own schedule,
  and a power cut or ISP outage takes the HR system down with it.
- **Failures are silent.** A dead tunnel looks exactly like nothing —
  the same lesson as [[watchtower-push-to-live|the frozen deploy loop]]:
  loops that die quietly need something that checks them. Here that
  watcher is the OS itself — the tunnel runs as a service that restarts
  on crash, and the containers restart with Docker.
- **Backups are nobody's problem but mine.** The database lives in a
  Docker volume on one disk — for an HR system, the sharpest edge of
  the whole setup. The mitigation: a nightly scheduled `pg_dump` lands
  in a second, cloud-synced location, with two weeks of retention.
- **Cloudflare sees the plaintext** and is a single dependency; fine at
  pilot stage, worth naming.

The reasoning: a pilot with one company and unproven revenue should
cheap out on exactly one thing — availability — while keeping the
security fundamentals real. Renting production infrastructure before
the product is proven buys uptime the pilot doesn't need yet with money
it hasn't earned.

## What I learned

A production *pilot* doesn't need production *infrastructure*. The
architecture is honest about its stage: one machine, one compose file,
zero hosting bill — but with real TLS, a hidden origin, encrypted
employee data, boot-time migrations, and nightly off-disk backups, so
graduating to a VPS later is a `docker compose up` on different
hardware, not a redesign.

Same tunnel trick as the portfolio — and the same class of surprise:
see [[nginx-https-downgrade-behind-cloudflare-tunnel|How my website quietly lost its security padlock]].

Related: [[watchtower-push-to-live|How my site updates itself minutes after I save my work]], [[spec-first-agentic-builds|Write the plan before the code]]
