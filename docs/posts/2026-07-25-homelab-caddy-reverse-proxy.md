---
date:
  created: 2026-07-25
categories:
  - Homelab
tags:
  - homelab
  - caddy
  - reverse-proxy
authors:
  - rubot
draft: true
---

# Home lab, part 4: reverse proxy migration, Nginx Proxy Manager to Caddy

[Part 3](../home-lab-part-3-from-proxmox-to-bare-metal-docker/) covered the move to bare-metal Docker, and mentioned in passing that the always-on server runs n8n, Dockhand, and Caddy. This post is about that last one — specifically, about what it replaced: Nginx Proxy Manager, which sat in front of everything before this. Where it lives on the network is nothing new; it's still on the Trusted VLAN, in the Secure zone, per [part 2](../home-lab-part-2-network-architecture-and-vlan-segmentation/) — nothing about the migration changed that.

<!-- more -->

## What NPM was for

Nginx Proxy Manager did its job well for a long time. It's Nginx underneath, but you never touch a config file directly — you add a proxy host through a web UI, point it at a host and port, tick a box for SSL, and Let's Encrypt certs get issued and renewed for you. For getting a home lab's first few services exposed behind HTTPS without learning Nginx's directive syntax, that's a genuinely good trade.

## Why I moved anyway

The thing that started to bother me wasn't a bug or an outage — NPM never gave me either. It was that the entire proxy configuration lived in a SQLite database behind a UI, not in anything I could put in git, diff, or read in five seconds to answer "what does this actually do." Every proxy host was a form I'd filled in once and then had to click back through if I wanted to check it.

Caddy inverted that. It's a single Go binary, and the entire configuration is a Caddyfile — plain text, one block per site, that you can version-control, diff, and read top to bottom without opening a browser. That fits the same instinct behind [dropping Proxmox](../home-lab-part-3-from-proxmox-to-bare-metal-docker/) for bare-metal Docker: fewer layers between me and the thing that's actually running, and a config I can reason about directly instead of through a management UI's idea of it.

The other selling point was automatic HTTPS by default rather than as a checkbox — Caddy requests and renews certs itself the moment a site block resolves, with no separate step to remember.

## NPM's UI vs. a Caddyfile

I didn't keep an export of the old NPM config — it lived in that SQLite database, not a file, which was rather the point of moving away from it. But the shape of the comparison holds regardless of the exact hosts involved. Where NPM represented each proxied service as a row in a database behind a form, the equivalent in Caddy is a block like this (domains and internal addresses below are illustrative, not what's actually configured):

```caddyfile
app.example.internal {
    reverse_proxy 10.20.5.10:8080
}

n8n.example.internal {
    reverse_proxy 10.20.5.11:5678
}
```

That's the entire proxy host, including TLS — no separate certificate step, no toggling a switch in a UI. Add a new service and it's a new block; change a backend port and it's a one-line diff instead of a click-through.

## What tripped me up

The Caddyfile syntax took longer to click than I expected, mostly because I came in assuming it would map fairly directly onto Nginx's `server`/`location` blocks and kept trying to think in those terms. It doesn't, and once I stopped forcing that mapping and worked through Caddy's own docs and a couple of tutorials instead, it came together quickly — the syntax is genuinely simpler than Nginx's once you're reading it on its own terms rather than translating.

Past that initial hump, the migration itself was uneventful: move services across one at a time, confirm each one resolves and gets a valid cert under Caddy, then decommission the matching NPM entry. No overlap issues, no downtime worth mentioning.

## Where things stand

Caddy now handles every proxied service on the Lab VLAN, running as one of the always-on containers alongside n8n and Dockhand on the 24/7 server from part 3. NPM is gone entirely — no lingering install kept around "just in case," since there's nothing left for it to do.
