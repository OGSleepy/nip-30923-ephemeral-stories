# NIP-XX: Ephemeral Stories — Kind 30923

A proposal for Instagram/Snapchat-style ephemeral media stories on Nostr. Short-lived. Media-first. Chronological.

---

## What is this?

Stories are parameterized replaceable events (`kind: 30923`) that contain a single photo or video, expire after 24 hours via NIP-40, and are displayed in a ring/avatar UI pattern similar to Snapchat and Instagram Stories.

This NIP defines the event structure, recommended client behavior, and how story feeds should be constructed.

---

## Event Structure

```json
{
  "kind": 30923,
  "content": "<optional caption, max 140 chars>",
  "created_at": 1700000000,
  "tags": [
    ["d", "<unique identifier, e.g. random hex or timestamp>"],
    ["url", "https://cdn.example.com/abc123.jpg"],
    ["m", "image/jpeg"],
    ["x", "<sha256 hash of the media file>"],
    ["expiration", "1700086400"],
    ["alt", "<caption or description for accessibility>"]
  ]
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique identifier. Makes the event parameterized replaceable (NIP-33). Clients SHOULD use a random value so each story is distinct. |
| `url` | Direct URL to the media file (image or video). MUST be a publicly accessible URL. |
| `m` | MIME type of the media (`image/jpeg`, `image/png`, `image/gif`, `video/mp4`, `video/webm`, etc.). |
| `x` | SHA-256 hash of the media file for integrity verification (BUD-02 compatible). |
| `expiration` | Unix timestamp after which the story SHOULD be considered expired and hidden (NIP-40). SHOULD be `created_at + 86400` (24 hours). |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `alt` | Text description of the media for accessibility and non-media clients. Mirrors `content` if a caption is present. |
| `size` | File size in bytes. |
| `dim` | Dimensions in `WxH` format (e.g. `1080x1920`). |
| `blurhash` | BlurHash string for placeholder rendering before media loads. |
| `thumb` | URL to a thumbnail image (useful for video stories). |

---

## Kind Range

`30923` falls in the `30000–39999` range: **parameterized replaceable events** (NIP-01).

- Each story with a unique `d` tag is a separate story (not a replacement).
- To replace/update a story, publish a new event with the same `d` tag.
- To delete a story, publish a kind `5` deletion event referencing the story's event ID (NIP-09).

---

## Expiration

Stories MUST include an `expiration` tag per NIP-40.

- Relays that support NIP-40 SHOULD stop serving events past their expiration.
- Clients MUST filter out expired stories regardless of relay behavior.
- The recommended story lifetime is **86400 seconds (24 hours)**.

```
"expiration": String(created_at + 86400)
```

---

## Media Hosting

Media files SHOULD be hosted on a Blossom server (BUD-02). The `x` tag MUST contain the SHA-256 hash of the uploaded file, which Blossom servers use as the content-addressed filename.

Clients SHOULD verify that `sha256(downloaded_file) === x_tag_value` before displaying media.

---

## Client Behavior

### Publishing

1. Upload the media to a Blossom server or compatible CDN.
2. Record the returned URL and SHA-256 hash.
3. Sign and publish a `kind: 30923` event to the user's NIP-65 write relays.
4. Include `expiration = created_at + 86400`.

### Displaying

1. Fetch `kind: 30923` events from the user's NIP-65 read relays.
2. Filter: only display events where `expiration > now()`.
3. Filter: only display events that have a valid `url` tag with a media MIME type (`image/*`, `video/*`, `image/gif`).
4. Group events by `pubkey` — each pubkey's stories form a single ring.
5. Display rings sorted: the current user's ring first, then unviewed rings, then viewed rings.
6. Within a ring, display stories ordered by `created_at` ascending.

### Story Viewer

- Auto-advance between stories in a ring after a fixed duration (recommended: 5s for photos, 15s for GIFs, video duration for video).
- Allow tap-left / tap-right navigation within a ring.
- Swiping down or pressing X closes the viewer.
- For video: mute by default, unmute on tap.

### Seen State

Clients SHOULD track which story event IDs have been viewed locally (not published to relays). There is no server-side seen receipt in this NIP. The `viewedStories` set SHOULD be persisted in client storage.

---

## Filtering

Clients MUST reject events that:
- Are past their `expiration` timestamp.
- Have no `url` tag.
- Have a `url` tag but no recognizable media MIME type in `m` (and no media extension in the URL).
- Have `m` values that are not `image/*` or `video/*`.

---

## Suggested Relay Filter

```json
{
  "kinds": [30923],
  "since": <now - 86400>,
  "limit": 200
}
```

Add `"authors": [<pubkeys>]` to fetch only followed users' stories.

---

## Example: Full Story Event

```json
{
  "id": "abc123...",
  "pubkey": "def456...",
  "kind": 30923,
  "content": "gm nostr ☀️",
  "created_at": 1700000000,
  "tags": [
    ["d", "story-1700000000-abc"],
    ["url", "https://blossom.primal.net/b1674191a88ec5cdd733e4240a81803105dc412d6c6708d53ab94fc248f4f553.jpg"],
    ["m", "image/jpeg"],
    ["x", "b1674191a88ec5cdd733e4240a81803105dc412d6c6708d53ab94fc248f4f553"],
    ["expiration", "1700086400"],
    ["alt", "gm nostr ☀️"],
    ["dim", "1080x1920"]
  ],
  "sig": "..."
}
```

---

## Relation to Other NIPs

| NIP | Relation |
|-----|---------|
| NIP-01 | Base protocol, kind ranges |
| NIP-09 | Story deletion via kind 5 |
| NIP-33 | Parameterized replaceable events (`d` tag) |
| NIP-40 | Expiration tag |
| NIP-65 | Relay list for publishing and discovery |
| BUD-02 | Blossom media uploads (recommended hosting) |

---

## Implementations

- [Flare](https://flarenos.pages.dev) — Snapchat-style story client for Nostr ([source](https://github.com/OGSleepy/Flare))

---

## Notes

- Clients that do not implement this NIP will ignore these events gracefully per NIP-01.
- Story replies are handled via NIP-17 encrypted DMs — a reply to a story is sent as a kind 14 rumor with the story URL embedded in the content.
- View-once snap DMs (media that disappears after being opened) are sent via NIP-17 with a `["view-once", "true"]` tag on the kind 14 rumor. This is not part of this NIP.
