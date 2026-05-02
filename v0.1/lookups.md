# Lookups

Post lookup by ID and relationship navigation. These endpoints use `GET` for direct reads, unlike [algorithmic queries](queries.md) which use `POST`.

Providers declare lookup support in the [discovery document](discovery.md). When the `lookups.posts` field is present, the post lookup and relationship endpoints described here are available.

## Post Lookup

Fetch a single post by provider-scoped ID.

### Request

```http
GET /api/v1/posts/{id}
Authorization: Bearer cf_live_abc123xyz
```

`{id}` is a provider-scoped integer identifier from the `id` field on [PostItem](posts.md#fields).

### Response

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requestId` | String | Yes | Unique request identifier |
| `post` | PostItem | Yes | The requested post. See [Posts](posts.md). The `id` field MUST be present. Nested `replyTo` and `quote` items follow the standard [nesting constraints](posts.md#replies) |

<details>
<summary>Example</summary>

```json
{
  "requestId": "req_lu789",
  "post": {
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
      "name": "Alice",
      "handle": "@alice@social.example",
      "url": "https://social.example/@alice",
      "identifiers": {
        "activitypub": "https://social.example/users/alice"
      }
    },
    "timestamp": "2026-02-28T09:15:00Z",
    "language": "en",
    "engagement": {
      "likes": 142,
      "reposts": 38,
      "replies": 12
    }
  }
}
```

</details>

### Errors

| Status | Error Code | When |
|--------|-----------|------|
| `400` | `bad_request` | `{id}` is not a valid integer |
| `404` | `not_found` | Post does not exist or has been deleted |

All standard errors from [Errors](errors.md) also apply (authentication, rate limiting, etc.).

## Relationship Navigation

Paginated lists of posts related to a parent post. These endpoints return results in the provider's default ordering (typically reverse chronological). No algorithm selection is needed.

### Replies

```http
GET /api/v1/posts/{id}/replies?limit=20
Authorization: Bearer cf_live_abc123xyz
```

Returns posts that are direct replies to the specified post.

### Quotes

```http
GET /api/v1/posts/{id}/quotes?limit=20
Authorization: Bearer cf_live_abc123xyz
```

Returns posts that quote the specified post.

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `limit` | Integer | No | Number of results to return. Range 1 to the provider's `maxResults` (see [Discovery](discovery.md)). Default 20. Over-limit values are silently clamped |
| `cursor` | String | No | Opaque cursor from a previous response for pagination |

### Response

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requestId` | String | Yes | Unique request identifier |
| `results` | Array | Yes | [PostItem](posts.md) results. Each item MUST include `id` |
| `pagination` | Object | Yes | Pagination info (same structure as [query responses](responses.md)) |
| `pagination.cursor` | String | No | Cursor for next page (`null` if last) |
| `pagination.cursorExpiresAt` | String | No | ISO 8601 expiry time for the cursor |
| `pagination.hasMore` | Boolean | Yes | Whether more results exist |

There is no `algorithm` field; these endpoints use provider-default ordering.

<details>
<summary>Example</summary>

```json
{
  "requestId": "req_rel456",
  "results": [
    {
      "id": 48291102,
      "url": "https://social.example/@bob/112399",
      "protocol": "activitypub",
      "identifiers": {
        "activitypub": "https://social.example/users/bob/statuses/112399"
      },
      "type": "reply",
      "content": "<p>Agreed — the renewable targets are ambitious but achievable.</p>",
      "text": "Agreed — the renewable targets are ambitious but achievable.",
      "author": {
        "name": "Bob",
        "handle": "@bob@social.example",
        "url": "https://social.example/@bob",
        "identifiers": {
          "activitypub": "https://social.example/users/bob"
        }
      },
      "timestamp": "2026-02-28T10:02:00Z",
      "language": "en",
      "engagement": {
        "likes": 23,
        "reposts": 2,
        "replies": 1
      }
    }
  ],
  "pagination": {
    "cursor": "eyJvIjoyMH0=",
    "cursorExpiresAt": "2026-03-02T16:00:00Z",
    "hasMore": true
  }
}
```

</details>

### Errors

| Status | Error Code | When |
|--------|-----------|------|
| `400` | `bad_request` | `{id}` is not a valid integer, or invalid query parameter |
| `404` | `not_found` | Parent post does not exist or has been deleted |

When the parent post exists but has no replies or quotes, the response is `"results": []` with `"hasMore": false`.

## Algorithmic Relationship Queries

Replies and quotes can also be ranked by an algorithm, using the same request format as [algorithmic queries](queries.md).

```http
POST /api/v1/posts/{id}/replies/{algorithm}
POST /api/v1/posts/{id}/quotes/{algorithm}
Content-Type: application/json
Authorization: Bearer cf_live_abc123xyz
```

The request body follows the standard [query format](queries.md): `filters`, `embedding`, `limit`, and `cursor` all work the same way.

The response uses the standard [query response](responses.md) envelope, including the `algorithm` field.

### Availability

An algorithmic relationship query is available when:

1. The provider declares `lookups.posts` in the [discovery document](discovery.md)
2. The relationship type (e.g. `replies`) is listed in `lookups.posts.relationships`
3. The algorithm name is listed in `lookups.posts.algorithms`

For example, if the discovery document declares:

```json
{
  "lookups": {
    "posts": {
      "relationships": ["replies", "quotes"],
      "algorithms": ["trending"]
    }
  }
}
```

Then `POST /api/v1/posts/{id}/replies/trending` and `POST /api/v1/posts/{id}/quotes/trending` are available.

### Example

```http
POST /api/v1/posts/48291037/replies/trending
Content-Type: application/json
Authorization: Bearer cf_live_abc123xyz
```

```json
{
  "filters": {
    "language": ["en"]
  },
  "limit": 20
}
```

Response follows the standard [query response](responses.md) format.
