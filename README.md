# NIP-30923: Ephemeral Stories

> 📸 A Nostr standard for Snapchat/Instagram-style ephemeral media stories.

---

## Overview

This repository proposes **kind `30923`** as a standard for ephemeral, media-first stories on Nostr — photos and videos that expire after 24 hours and are displayed in the familiar story ring UI pattern.

The spec lives in [`nip-ephemeral-stories.md`](./nip-ephemeral-stories.md).

---

## Why?

Nostr has no standard for ephemeral social stories. Apps like [Flare](https://flarenos.pages.dev) have implemented this pattern but without an agreed-upon spec, different clients will use different kinds, different tags, and incompatible formats.

This NIP aims to fix that — so that stories posted from one client can be read by any other client that implements the standard.

---

## The Short Version

- **Kind:** `30923` (parameterized replaceable, NIP-33)
- **Expiry:** `expiration` tag set to `created_at + 86400` (NIP-40)
- **Media:** hosted on Blossom (BUD-02), referenced by `url` + `m` (MIME type) + `x` (SHA-256)
- **Caption:** optional, in `content` (max 140 chars)
- **Discovery:** via NIP-65 relay list
- **Deletion:** kind `5` event referencing the story's event ID (NIP-09)

```json
{
  "kind": 30923,
  "content": "gm nostr ☀️",
  "tags": [
    ["d", "story-<unique-id>"],
    ["url", "https://blossom.primal.net/<hash>.jpg"],
    ["m", "image/jpeg"],
    ["x", "<sha256>"],
    ["expiration", "<created_at + 86400>"],
    ["alt", "gm nostr ☀️"]
  ]
}
```

---

## Why `30923`?

`30923` is in the `30000–39999` range — **parameterized replaceable events** (NIP-01). This is intentional:

- The `d` tag makes each story uniquely addressable
- Publishing a new event with the same `d` tag replaces/updates that story
- Publishing events with different `d` tags creates multiple independent stories per user
- The word "ephemeral" in this context refers to the **product behavior** (stories disappear after 24h via NIP-40), not the Nostr protocol concept of ephemeral events (kind `20000–29999`)

---

## Implementations

| Client | Platform | Link |
|--------|----------|------|
| **Flare** | Web / PWA | [flarenos.pages.dev](https://flarenos.pages.dev) · [source](https://github.com/OGSleepy/Flare) |

Building something? Open a PR to add it here.

---

## Status

This is a **draft proposal**. It has not been submitted to the official [nostr-protocol/nips](https://github.com/nostr-protocol/nips) repository yet. Feedback and discussion welcome via Issues.

---

## Related NIPs

| NIP | Role |
|-----|------|
| NIP-01 | Base protocol, kind ranges |
| NIP-09 | Story deletion (kind 5) |
| NIP-17 | Story replies via encrypted DM |
| NIP-33 | Parameterized replaceable events |
| NIP-40 | Expiration timestamp |
| NIP-65 | Relay list for publishing |
| BUD-02 | Blossom media uploads |

---

## Contributing

Open an issue or PR. The goal is to get this widely adopted so Nostr has a real stories standard.

---

Built on [Nostr](https://nostr.com) · First implemented in [Flare](https://flarenos.pages.dev) by [@OGSleepy](https://github.com/OGSleepy)
