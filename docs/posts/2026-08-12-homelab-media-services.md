---
date:
  created: 2026-08-12
categories:
  - Homelab
tags:
  - homelab
  - media
  - self-hosting
authors:
  - rubot
draft: false
---

# Home lab, part 4: self-hosted media services

[Part 3](../home-lab-part-3-from-proxmox-to-bare-metal-docker/) covered the compute layer — two servers, bare metal, Docker straight on top. This post covers what's actually running on the day-only box for media: Plex, Navidrome, Kavita, and Audiobookshelf.

<!-- more -->

Four apps for what could plausibly be "media" as a single category is more than it sounds like it needs to be, and the instinct to consolidate is a real one — fewer containers, fewer things to patch, one login instead of four. But the thing that actually determines whether a media app is any good isn't the format it reads, it's the job it's doing: playing a movie on a TV, downloading an album for a flight with no signal, casting a podcast-shaped audiobook to a kitchen speaker, and reading a comic on a tablet are different jobs even when two of them technically involve "audio". Each app below earned its place by being the best tool for one specific job, not one format.

## Movies and TV: Plex

Plex handles the video library, and the honest reason is thirteen years of history plus a lifetime license bought years ago. If I were starting from nothing today, I'd probably reach for Jellyfin instead — but "probably better if starting over" isn't the same argument as "worth migrating thirteen years of watch history and metadata away from a license I've already paid for." Sometimes the right tool is the incumbent, not because it's still winning on merit alone, but because switching has a cost the alternative would have to clear, and it doesn't clear it by enough.

## Music, split by what you're doing with it

Music is the one place a single format ended up needing two apps, because "listen to music" turned out to be two different jobs.

**PlexAmp** handles offline listening — it downloads albums to the phone for playback with no connection, which is exactly what you want for a flight or a dead spot in the countryside. What it doesn't do is cast: a track downloaded into PlexAmp plays from local storage on that device and stays there, it doesn't go to a speaker across the room.

For that, **Navidrome** with **Symfonium** as the client covers casting and shared listening — the OpenSubsonic API that Navidrome speaks is what lets a client like Symfonium throw audio at a device the way PlexAmp can't. Navidrome also does shared playlists and per-user listening stats, which matters with two of us on the same library. It was the obvious pick for the role: it's the most popular self-hosted option in that category, and one of its longer-standing alternatives, Airsonic, has since gone closed-source.

So PlexAmp and Navidrome aren't two apps competing for the same job — one is for taking music with you, the other is for playing it out loud where you are.

## Comics, manga, and ebooks: Kavita

Kavita reads both comics/manga and ebooks, which on its own is the pitch: the alternative was running two separate apps — something like Komga for comics and Calibre-web for ebooks — to cover the same ground Kavita covers alone. Beyond consolidating two apps into one, Kavita's UI is simply more polished than Calibre-web's, which made it an easy call even setting the two-apps-versus-one argument aside.

## Audiobooks: Audiobookshelf

Audiobookshelf is the least contested decision in this stack — it's the clear community favourite for self-hosted audiobooks, it has a dedicated mobile app, and there wasn't a real alternative worth weighing it against. Sometimes the design tree doesn't branch; the obvious answer is obvious for a reason.

## Where the libraries actually live

The QNAP NAS from [Part 1](../home-lab-part-1-hardware-and-physical-setup/) is the source of truth for every media file and where backups live, but it's low-powered — not something you want four services doing transcoding and indexing against directly. So the libraries are mirrored onto the day-only server from Part 3, and that's what Plex, Navidrome, Kavita, and Audiobookshelf actually run against and serve from. The NAS stays the archive; the server is the one doing the work.

All four apps sit on the Lab VLAN and are reachable only from inside the network right now — nothing here is exposed externally yet.

## The stack at a glance

| App | Job | Client |
|---|---|---|
| Plex | Movies and TV | Plex apps |
| Plex (PlexAmp) | Offline music downloads | PlexAmp |
| Navidrome | Music streaming and casting, shared playlists | Symfonium |
| Kavita | Comics, manga, ebooks | Web |
| Audiobookshelf | Audiobooks | Audiobookshelf app |

## Coming up next

Part 5 covers the rest of that NAS-to-day-server relationship — why the NAS doesn't run these apps itself, and what else the day server has quietly taken on.
