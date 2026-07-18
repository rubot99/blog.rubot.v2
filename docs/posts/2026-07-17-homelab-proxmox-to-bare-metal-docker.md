---
date:
  created: 2026-07-17
categories:
  - Homelab
tags:
  - homelab
  - docker
  - proxmox
authors:
  - rubot
draft: true
---

# Home lab, part 3: from Proxmox to bare-metal Docker

[Part 2](../home-lab-part-2-network-architecture-and-vlan-segmentation/) covered the network the Lab VLAN sits on. This post covers what's actually running inside it — and a decision I made and then partly reversed: standing up Proxmox on both servers, then tearing it back down in favour of running Docker straight on bare metal.

<!-- more -->

## What Proxmox is for

Proxmox earns its keep when one physical host needs to run several VMs that don't trust each other, need genuinely different kernels or OSes, or need to be snapshotted and rolled back independently of each other. That's a real problem, and Proxmox solves it well — a hypervisor layer, a web UI for lifecycle management, per-VM snapshots, live migration if you've got the cluster for it.

None of that was the problem I had.

## The layer that wasn't buying me anything

I had two servers. I put Proxmox on both. And on each one, I created exactly one VM.

That's not a criticism of Proxmox — it's a description of a mismatch. A hypervisor pays for itself by amortizing its overhead across multiple guests. With a 1:1 mapping of host to VM, I was paying full price — a second OS to patch, a second layer to troubleshoot through, a web UI and API surface to secure and keep up to date — for isolation between the VM and nothing else, because there was nothing else on the box to isolate it from. I'd built the enterprise-shaped solution for a problem I didn't have.

## What I actually lost by dropping it

Less than I expected. The obvious objection to going bare metal is losing VM-level snapshots and easy rollback — but I'd never set up backup infrastructure for Proxmox to write those snapshots to, so in practice that safety net didn't exist before either. I wasn't giving up something I was using; I was giving up something I'd still have needed to build to make Proxmox's case for itself.

## What I gained

- **One OS layer, not two.** Patching, log inspection, and troubleshooting all happen directly on the host instead of through a hypervisor console into a guest.
- **A cleaner credential footprint.** Every Proxmox install is another set of authentication to track — host, hypervisor UI, guest — and that had quietly metastasized across enough Bitwarden entries to be its own minor chore. Bare metal collapsed that back down to one set of credentials per box.
- **Simpler scheduling.** One of the two servers only needs to run during the day — it hosts Plex and a few other things nobody's using at 2am, so it shuts down overnight on a cron job. On Proxmox that's shutting down the VM, then the hypervisor, in sequence. On bare metal it's one machine going to sleep and waking up on schedule — same outcome, one fewer handoff to get wrong.

## Where things stand

Both servers run Ubuntu 26.04 LTS with Docker directly on top — no hypervisor in between. One runs Plex and the day-only services on the cron schedule above. The other runs 24/7: n8n, Dockhand, and Caddy.

## Proxmox isn't gone, just not here yet

None of this is an argument that Proxmox was the wrong tool in general — it's that it was the wrong tool for two boxes each running a single VM. The case for it comes back the moment the job changes: a dedicated host for spinning up disposable VMs, LXC containers, and unfamiliar OSes to poke at, sitting in its own sandboxed corner of the Lab VLAN where breaking something is the point. That's a future host, not a retrofit onto the two that are working fine as they are — and when it lands, it'll be because I need multiple isolated guests on one box again, not because a hypervisor felt like the "proper" way to run a home lab.

## Coming up next

Part 4 covers the reverse proxy sitting in front of these services — what's changing there and why.
