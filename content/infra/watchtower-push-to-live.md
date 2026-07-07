---
title: "How my site updates itself minutes after I save my work"
tags:
  - docker
  - ci
  - deployment
  - infra
date: 2026-07-06
---

> **In plain terms:** When I finish a change to my website, I don't upload
> anything or touch the server. Saving my work triggers an automated
> assembly line: the change is packaged up, and a small watcher program on
> my server notices the new package within a minute and swaps it in. This
> note describes that assembly line — and the two failures it hid from me,
> both of which broke things while everything *looked* green.

The deploy loop for my self-hosted portfolio: GitHub Actions (the
automated assembly line) builds a Docker image — a sealed, ready-to-run
package of the whole site — on every push to `main` and publishes it to a
registry; on the server, [Watchtower](https://containrrr.dev/watchtower/)
polls that registry every 60s and restarts the container when a new image
appears. The server never checks out the repo or runs a build —
`docker compose up` on any machine is a full deploy target.

```yaml
web:
  image: ghcr.io/ariefrsee/portfolio:latest
watchtower:
  image: containrrr/watchtower
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
```

Push → CI → registry → live, no SSH step. Two footguns cost me a debugging
session each:

## Footgun 1: build-time env vars don't exist in CI

With a static export, `NEXT_PUBLIC_*` values are **baked in at build
time**. My local builds had them; the CI workflow didn't pass them as
build-args. Result: CI-built images deployed green — working site, working
build — with a contact form that silently posted to nothing. The pipeline
being the *second* way the same site gets built is exactly where this class
of bug lives: everything that works locally and isn't in the workflow file
is broken in production.

## Footgun 2: Watchtower's Docker client is ancient

Watchtower's bundled client speaks Docker API 1.25; a current daemon
rejects anything below 1.40. Every poll cycle errored **before ever
reaching the registry** — so auto-deploy just quietly stopped happening.
No crash, container healthy, deploys frozen. The fix is one env var:

```yaml
environment:
  - DOCKER_API_VERSION=1.44
```

## What I learned

Both failures share a shape: the pipeline *runs* and the failure is
invisible from the outside. After wiring any auto-deploy loop, verify the
loop itself — push a trivial visible change and watch it arrive, and check
the poller's logs once, not never.

Related: [[nginx-https-downgrade-behind-cloudflare-tunnel|How my website quietly lost its security padlock]], [[known-ci-footguns|Teaching AI assistants to remember past mistakes]]
