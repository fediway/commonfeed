# Link Results

Result items for `links` resources. Each item represents a link with preview metadata and usage statistics.

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | String | Yes | Canonical URL of the linked page |
| `title` | String | Yes | Page title (from Open Graph / oEmbed) |
| `description` | String | Yes | Page description (from Open Graph / oEmbed) |
| `type` | String | Yes | oEmbed type: `link`, `photo`, `video`, `rich` |
| `image` | ImageObject | No | Preview image for the linked page (see [ImageObject](posts.md#imageobject)) |
| `providerName` | String | No | Site name (e.g. "Example News") |
| `authorName` | String | No | Article author name |
| `embedHtml` | String | No | oEmbed HTML for video/rich embeds |
| `embedUrl` | String | No | Embeddable player URL |
| `embedWidth` | Integer | No | Embed width in pixels |
| `embedHeight` | Integer | No | Embed height in pixels |
| `language` | String | No | Content language |
| `publishedAt` | String | No | ISO 8601 article publish date |
| `favicon` | ImageObject | No | Site favicon (see [ImageObject](posts.md#imageobject)) |
| `postCount` | Integer | Yes | Total posts sharing this link in the time window |
| `accountCount` | Integer | Yes | Unique authors sharing this link in the time window |

The link metadata fields (`url` through `publishedAt`) are the same shape as the [Link Object](posts.md#link-object) on post results. The Link Result adds usage statistics (`postCount`, `accountCount`).

<details>
<summary>Example</summary>

```json
{
  "url": "https://example.com/article",
  "title": "Example Article",
  "description": "A great article about climate policy",
  "type": "link",
  "image": {
    "sizes": {
      "small": {
        "url": "https://cdn.provider.example/links/og_image_s.webp",
        "width": 640,
        "height": 336,
        "mimeType": "image/webp",
        "fileSize": 32000
      },
      "large": {
        "url": "https://cdn.provider.example/links/og_image.webp",
        "width": 1200,
        "height": 630,
        "mimeType": "image/webp",
        "fileSize": 96000
      }
    },
    "blurhash": "LEHV6nWB2yk8pyo0adR*.7kCMdnj"
  },
  "providerName": "Example News",
  "authorName": "Alice Johnson",
  "favicon": {
    "sizes": {
      "large": {
        "url": "https://cdn.provider.example/favicons/abc123.webp",
        "width": 32,
        "height": 32,
        "mimeType": "image/webp"
      }
    }
  },
  "language": "en",
  "publishedAt": "2026-02-27T10:00:00Z",
  "postCount": 42,
  "accountCount": 15
}
```

</details>
