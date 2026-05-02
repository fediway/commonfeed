# Post Results

Result items for `posts` resources. Each item represents a piece of content from the fediverse.

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | Integer | No | Provider-scoped identifier. Unique within this provider. Providers SHOULD include this on all top-level result items. MUST be present on items returned from [lookup and relationship endpoints](lookups.md). MAY be absent on nested items (`replyTo`, `quote`) |
| `url` | String | Yes | Web-viewable link to the original content (always `https://`) |
| `protocol` | String | Yes | Source protocol: `activitypub`, `atproto`, `nostr` |
| `identifiers` | Object | Yes | Protocol-native identifiers. Keys are protocol names, values are the identifier for that protocol. Contains at minimum the native protocol's identifier. May include additional identifiers if the content provider ingested bridged copies |
| `type` | String | Yes | `post`, `article`, `reply` |
| `content` | String | Yes | HTML content. Provider normalizes from source format (AT Protocol facets, Nostr plaintext, etc.) |
| `text` | String | Yes | Plain text version of the content (HTML stripped, no formatting) |
| `author` | Object | Yes | Author info (see below) |
| `timestamp` | String | Yes | ISO 8601 publication time |
| `language` | String | No | ISO 639-1 language code |
| `sensitive` | Boolean | No | Media sensitivity flag. Consumers SHOULD blur media when `true` (default `false`) |
| `contentWarning` | String | No | Content warning text (e.g. "spoiler: movie ending"). Absent if none. Independent of `sensitive`: a post can have CW text without sensitive media, or sensitive media without CW text |
| `media` | Array | No | Attached media (absent if none, see below) |
| `link` | Object | No | Primary link preview (absent if none, see below) |
| `engagement` | Object | No | Approximate engagement counts (see below) |
| `replyTo` | PostItem | No | The parent post this is a reply to, inline as a full PostItem (absent if not a reply, see [Replies](#replies)) |
| `quote` | PostItem | No | The quoted post, inline as a full PostItem (absent if not a quote post, see [Quoted Posts](#quoted-posts)) |
| `tags` | Array | No | Hashtags used in the content (absent if none, see below) |
| `emojis` | Array | No | Custom emojis used in the content (absent if none, see below) |
| `context` | String | No | Opaque per-item context token. When present, consumers SHOULD pass this value back to the provider with engagement signals for the item. Providers use this for feedback loops (e.g. closing the loop between recommendation and observed engagement). Consumers MUST NOT interpret, parse, or make decisions based on the contents of this field |

Engagement counts are approximate and may be stale. Consumers SHOULD NOT treat them as authoritative.

### Protocol Identifier Formats

`identifiers` keys are protocol names; values are the canonical identifier in that protocol's native form. Format depends on whether the identifier refers to a status (post) or an actor (author):

| Protocol | Status (post) | Actor (author) |
|----------|---------------|----------------|
| `activitypub` | Post URI (the AP object's `id`, e.g. `https://social.example/users/alice/statuses/112345`) | Actor URI (the AP actor's `id`, e.g. `https://social.example/users/alice`) |
| `atproto` | AT URI (`at://did:plc:.../app.bsky.feed.post/...`) | DID (`did:plc:...` or `did:web:...`) |
| `nostr` | `nostr:<64-char lowercase hex event ID>` | `nostr:<64-char lowercase hex pubkey>` |

Providers MAY include identifiers from multiple protocols on the same item when content has been ingested via bridged copies (e.g. a Bluesky post bridged via Bridgy Fed will have both `atproto` and `activitypub` keys).

Because identifiers are protocol-canonical, consumers querying multiple providers MAY use them to deduplicate items returned by more than one.

<details>
<summary>Example</summary>

```json
{
  "id": 48291037,
  "url": "https://social.example/@alice/112345",
  "protocol": "activitypub",
  "identifiers": {
    "activitypub": "https://social.example/users/alice/statuses/112345"
  },
  "type": "post",
  "content": "<p>New EU climate policy looks promising for renewable energy targets.</p>",
  "text": "New EU climate policy looks promising for renewable energy targets.",
  "author": {
    "name": "Alice :verified:",
    "handle": "@alice@social.example",
    "url": "https://social.example/@alice",
    "identifiers": {
      "activitypub": "https://social.example/users/alice"
    },
    "avatar": {
      "sizes": {
        "small": {
          "url": "https://cdn.provider.example/avatars/7283910274839_s.webp",
          "width": 128,
          "height": 128,
          "mimeType": "image/webp",
          "fileSize": 4200
        },
        "large": {
          "url": "https://cdn.provider.example/avatars/7283910274839.webp",
          "width": 400,
          "height": 400,
          "mimeType": "image/webp",
          "fileSize": 18400
        }
      },
      "blurhash": "L5H2EC=PM+yV0g-mq.wG9c010J}t"
    },
    "emojis": [
      {
        "shortcode": "verified",
        "url": "https://cdn.provider.example/emoji/verified.png",
        "staticUrl": "https://cdn.provider.example/emoji/verified_static.png"
      }
    ]
  },
  "timestamp": "2026-02-28T09:15:00Z",
  "language": "en",
  "sensitive": false,
  "media": [
    {
      "type": "image",
      "alt": "EU parliament building with solar panels",
      "image": {
        "sizes": {
          "small": {
            "url": "https://cdn.provider.example/media/8291037461829_s.webp",
            "width": 640,
            "height": 427,
            "mimeType": "image/webp",
            "fileSize": 38200
          },
          "medium": {
            "url": "https://cdn.provider.example/media/8291037461829_m.webp",
            "width": 1280,
            "height": 853,
            "mimeType": "image/webp",
            "fileSize": 112000
          },
          "large": {
            "url": "https://cdn.provider.example/media/8291037461829.webp",
            "width": 2048,
            "height": 1365,
            "mimeType": "image/webp",
            "fileSize": 287000
          }
        },
        "blurhash": "LEHV6nWB2yk8pyo0adR*.7kCMdnj",
        "focalPoint": [0.0, 0.2]
      },
      "original": { "width": 3200, "height": 2133 }
    }
  ],
  "link": {
    "url": "https://example.com/eu-climate-policy",
    "title": "EU Climate Policy: What It Means for Renewable Energy",
    "description": "An analysis of the new EU climate targets and their impact on the renewable energy sector.",
    "type": "link",
    "image": {
      "sizes": {
        "small": {
          "url": "https://cdn.provider.example/links/9182736450123_s.webp",
          "width": 640,
          "height": 336,
          "mimeType": "image/webp",
          "fileSize": 32000
        },
        "large": {
          "url": "https://cdn.provider.example/links/9182736450123.webp",
          "width": 1200,
          "height": 630,
          "mimeType": "image/webp",
          "fileSize": 96000
        }
      },
      "blurhash": "LEHV6nWB2yk8pyo0adR*.7kCMdnj"
    },
    "providerName": "Example News"
  },
  "engagement": {
    "likes": 142,
    "reposts": 38,
    "replies": 12
  }
}
```

</details>

## ImageObject

A provider-served image with named size variants. This type is used everywhere an image appears in the spec: media attachments, avatars, link preview images, and video posters. All URLs are served by the provider; consumers display them directly without fetching from origin servers.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sizes` | Object | Yes | Named size variants (see below) |
| `blurhash` | String | No | BlurHash placeholder string |
| `focalPoint` | Array | No | Crop hint as `[x, y]` floats, each -1.0 to 1.0. `[0, 0]` is center, x increases rightward, y increases upward. Consumers SHOULD use this to position cropped images |

### Sizes Object

A map of named size variants. Each key is a size name, each value is a size variant. Providers MUST include at least `large`.

Well-known size names:

| Name | Description |
|------|-------------|
| `small` | Smallest variant, optimized for contexts with many images on screen |
| `medium` | Mid-range variant, optimized for single-image focused viewing |
| `large` | Highest resolution variant the provider serves |

Providers SHOULD NOT upscale. If the source is smaller than a tier's target dimensions, the provider MAY omit that tier or serve at source dimensions.

Providers MAY include additional custom size names. Consumers MUST ignore unrecognized size names.

### Size Variant

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | String | Yes | Direct URL to this variant, served by the provider |
| `width` | Integer | No | Width in pixels |
| `height` | Integer | No | Height in pixels |
| `mimeType` | String | No | MIME type (e.g. `image/webp`, `video/mp4`) |
| `fileSize` | Integer | No | File size in bytes |

Providers SHOULD include `width`, `height`, and `fileSize` on all size variants so consumers can layout before loading and estimate bandwidth.

### Image Recommendations

The spec does not mandate specific dimensions or formats. The following recommendations help providers produce consistent, high-quality results:

| Category | Variant | Recommended max | Expected file size | Format |
|----------|---------|----------------|-------------------|--------|
| Media images | `small` | 640px | 20–80 KB | WebP |
| | `medium` | 1280px | 60–200 KB | WebP |
| | `large` | 2560px | 150–500 KB | WebP |
| Avatars | `small` | 128px | 2–8 KB | WebP |
| | `large` | 400px | 5–20 KB | WebP |
| Headers | `small` | 640px | 20–80 KB | WebP |
| | `large` | 1500px | 60–200 KB | WebP |
| Link preview images | `small` | 640px | 20–80 KB | WebP |
| | `large` | 1280px | 60–200 KB | WebP |
| Video posters | `small` | 640px | 20–80 KB | WebP |
| | `large` | 1280px | 60–200 KB | WebP |

All dimensions refer to the longest side. Providers SHOULD preserve aspect ratio.

General guidance:

- Providers SHOULD serve images in WebP format for size efficiency
- Providers SHOULD strip all metadata (EXIF, XMP, GPS coordinates) from served images
- Providers SHOULD NOT upscale images; if the source is smaller than the target, serve at source dimensions
- The `large` variant is always required; `small` and `medium` are optional but RECOMMENDED for media images

## Author Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Display name |
| `handle` | String | Yes | Human-readable handle. Format varies by protocol: `@user@domain` (ActivityPub), `user.bsky.social` (AT Protocol), NIP-05 or truncated `npub1...` (Nostr) |
| `url` | String | Yes | Web-viewable profile link (always `https://`) |
| `identifiers` | Object | Yes | Protocol-native author identifiers. Same structure as post `identifiers` |
| `avatar` | ImageObject | No | Author's avatar image. Absent if the author has no avatar |
| `header` | ImageObject | No | Author's header/banner image. Absent if the author has no header |
| `emojis` | Array | No | Custom emojis used in the author's display name (absent if none) |

## Engagement Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `likes` | Integer | No | Like/favourite count |
| `reposts` | Integer | No | Repost/boost count |
| `replies` | Integer | No | Reply count |

## Tag Object

Each item in the `tags` array. Represents a hashtag used in the post content. The `name` is normalized (lowercase, no `#` prefix). Consumers construct display URLs from their own context.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Normalized tag name (lowercase, without `#`) |

## Replies

When a post is a reply, the `replyTo` field contains the parent post as a PostItem. This gives consumers everything needed to render reply context (e.g. showing the parent post above the reply in a feed) without additional requests.

The parent PostItem follows the standard PostItem schema with the following constraints:

- `id`: MAY be absent
- `replyTo`: always absent (no recursive nesting; max depth is 1)
- `quote`: always absent
- `context`: always absent

If the parent post is unavailable (deleted, not yet resolved), providers MUST omit `replyTo` entirely rather than serving partial data.

Providers MAY omit heavy optional fields (`media`, `link`, `emojis`) on the nested parent to manage payload size. At minimum, providers SHOULD include `url`, `author`, `content`, `text`, `timestamp`, and `engagement`.

## Quoted Posts

When a post quotes another post, the `quote` field contains the full quoted post as a PostItem. This gives consumers everything needed to render an inline quote card without additional requests.

The quoted PostItem follows the standard PostItem schema with the following constraints:

- `id`: MAY be absent
- `quote`: always absent (no recursive nesting; max depth is 1)
- `replyTo`: always absent
- `context`: always absent

If the quoted post is unavailable (deleted, not yet resolved), providers MUST omit `quote` entirely rather than serving partial data.

<details>
<summary>Example</summary>

```json
{
  "url": "https://social.example/@alice/112345",
  "protocol": "activitypub",
  "identifiers": {
    "activitypub": "https://social.example/users/alice/statuses/112345"
  },
  "type": "post",
  "content": "<p>This is a great take 👇</p>",
  "text": "This is a great take 👇",
  "author": {
    "name": "Alice",
    "handle": "@alice@social.example",
    "url": "https://social.example/@alice",
    "identifiers": {
      "activitypub": "https://social.example/users/alice"
    }
  },
  "timestamp": "2026-02-28T09:15:00Z",
  "engagement": {
    "likes": 42,
    "reposts": 10,
    "replies": 3
  },
  "quote": {
    "url": "https://social.example/@bob/98765",
    "protocol": "activitypub",
    "identifiers": {
      "activitypub": "https://social.example/users/bob/statuses/98765"
    },
    "type": "post",
    "content": "<p>Hot take: tabs are better than spaces.</p>",
    "text": "Hot take: tabs are better than spaces.",
    "author": {
      "name": "Bob",
      "handle": "@bob@social.example",
      "url": "https://social.example/@bob",
      "identifiers": {
        "activitypub": "https://social.example/users/bob"
      }
    },
    "timestamp": "2026-02-27T14:00:00Z",
    "language": "en",
    "engagement": {
      "likes": 128,
      "reposts": 45,
      "replies": 67
    }
  }
}
```

</details>

## Link Object

The primary link referenced in the post, with preview metadata. Absent if the post contains no links or if the link's metadata has not yet been fetched. At most one per post.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | String | Yes | Canonical URL of the linked page |
| `title` | String | Yes | Page title (from Open Graph / oEmbed) |
| `description` | String | Yes | Page description (from Open Graph / oEmbed) |
| `type` | String | Yes | oEmbed type: `link`, `photo`, `video`, `rich` |
| `image` | ImageObject | No | Preview image for the linked page |
| `providerName` | String | No | Site name (e.g. "Example News") |
| `authorName` | String | No | Article author name |
| `embedHtml` | String | No | oEmbed HTML for video/rich embeds |
| `embedUrl` | String | No | Embeddable player URL |
| `embedWidth` | Integer | No | Embed width in pixels |
| `embedHeight` | Integer | No | Embed height in pixels |
| `language` | String | No | Content language |
| `publishedAt` | String | No | ISO 8601 article publish date |
| `favicon` | ImageObject | No | Site favicon |

## Media Object

Each item in the `media` array. All URLs are provider-served; the consumer displays them directly without fetching from origin servers.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | String | Yes | `image`, `video`, `audio` |
| `alt` | String | No | Alt text / description. Providers SHOULD include this when the source provides it |
| `image` | ImageObject | Conditional | The image data (required when `type` is `image`, absent otherwise) |
| `original` | Object | No | Source dimensions before processing (see below). Providers SHOULD include this when known (image, video) |
| `sizes` | Object | Conditional | Named size variants for video/audio (required when `type` is `video` or `audio`, absent for `image`). Same structure as ImageObject sizes but contains video/audio files |
| `duration` | Float | No | Duration in seconds (video, audio) |
| `poster` | ImageObject | No | Preview image shown before playback. Providers SHOULD include this for video |
| `animated` | Boolean | No | Content is a short animation (e.g. converted from GIF). Consumers SHOULD autoplay and mute. Default `false` (video only) |
| `loop` | Boolean | No | Content should loop continuously. Consumers SHOULD loop playback. Default `false` (video only) |

### Original Object

Source dimensions before any processing. Lets consumers know the true resolution and aspect ratio.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `width` | Integer | Yes | Source width in pixels |
| `height` | Integer | Yes | Source height in pixels |

<details>
<summary>Example: image with three sizes</summary>

```json
{
  "type": "image",
  "alt": "A sunset over the ocean",
  "image": {
    "sizes": {
      "small": {
        "url": "https://cdn.provider.example/media/1234_s.webp",
        "width": 640, "height": 427,
        "mimeType": "image/webp",
        "fileSize": 42000
      },
      "medium": {
        "url": "https://cdn.provider.example/media/1234_m.webp",
        "width": 1280, "height": 853,
        "mimeType": "image/webp",
        "fileSize": 128000
      },
      "large": {
        "url": "https://cdn.provider.example/media/1234.webp",
        "width": 2560, "height": 1707,
        "mimeType": "image/webp",
        "fileSize": 340000
      }
    },
    "blurhash": "LEHV6nWB2yk8pyo0adR*.7kCMdnj",
    "focalPoint": [0.0, 0.3]
  },
  "original": { "width": 4000, "height": 2667 }
}
```

</details>

<details>
<summary>Example: animated GIF as video</summary>

```json
{
  "type": "video",
  "alt": "Cat falling off a table",
  "animated": true,
  "loop": true,
  "duration": 3.2,
  "original": { "width": 480, "height": 360 },
  "poster": {
    "sizes": {
      "small": {
        "url": "https://cdn.provider.example/media/5678_poster_s.webp",
        "width": 480, "height": 360,
        "mimeType": "image/webp",
        "fileSize": 18000
      },
      "large": {
        "url": "https://cdn.provider.example/media/5678_poster.webp",
        "width": 480, "height": 360,
        "mimeType": "image/webp",
        "fileSize": 18000
      }
    },
    "blurhash": "LKO2?U%2Tw=w]~RBVZRi};RPxuwH"
  },
  "sizes": {
    "large": {
      "url": "https://cdn.provider.example/media/5678.mp4",
      "width": 480, "height": 360,
      "mimeType": "video/mp4",
      "fileSize": 524000
    }
  }
}
```

</details>

<details>
<summary>Example: video with poster</summary>

```json
{
  "type": "video",
  "alt": "Conference keynote highlight",
  "duration": 45.0,
  "original": { "width": 1920, "height": 1080 },
  "poster": {
    "sizes": {
      "small": {
        "url": "https://cdn.provider.example/media/9012_poster_s.webp",
        "width": 640, "height": 360,
        "mimeType": "image/webp",
        "fileSize": 32000
      },
      "large": {
        "url": "https://cdn.provider.example/media/9012_poster.webp",
        "width": 1280, "height": 720,
        "mimeType": "image/webp",
        "fileSize": 84000
      }
    },
    "blurhash": "LEHV6nWB2yk8pyo0adR*.7kCMdnj"
  },
  "sizes": {
    "large": {
      "url": "https://cdn.provider.example/media/9012.mp4",
      "width": 1920, "height": 1080,
      "mimeType": "video/mp4",
      "fileSize": 12400000
    }
  }
}
```

</details>

<details>
<summary>Example: audio</summary>

```json
{
  "type": "audio",
  "alt": "Podcast episode: Climate Change in 2026",
  "duration": 1845.0,
  "sizes": {
    "large": {
      "url": "https://cdn.provider.example/media/3456.mp3",
      "mimeType": "audio/mpeg",
      "fileSize": 29520000
    }
  }
}
```

</details>

## Emoji Object

Custom emojis used in content or display names. Consumers render these by replacing `:shortcode:` occurrences with the emoji image. All URLs are provider-served.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `shortcode` | String | Yes | Emoji shortcode (without colons) |
| `url` | String | Yes | URL to the emoji image, served by the provider (may be animated) |
| `staticUrl` | String | No | URL to a static (non-animated) version, served by the provider |
