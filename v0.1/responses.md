# Responses

All query endpoints return a common response envelope. The shape of each result item depends on the resource type being queried (see [Posts](posts.md), [Tags](tags.md), and [Links](links.md)).

## Response (200)

```json
{
  "requestId": "req_abc123",
  "results": [ ... ],
  "algorithm": "commonfeed-trending-v1",
  "pagination": {
    "cursor": "eyJvIjoyMH0=",
    "cursorExpiresAt": "2026-03-02T16:00:00Z",
    "hasMore": true
  }
}
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requestId` | String | Yes | Unique request identifier for debugging and support |
| `results` | Array | Yes | Result items. Shape depends on the resource type: [Post](posts.md) for `posts`, [Tag](tags.md) for `tags`, [Link](links.md) for `links` |
| `algorithm` | String | Yes | Provider-defined algorithm identifier. This is an opaque string, not necessarily the same as the capability's `algorithm` name from [Discovery](discovery.md). Consumers SHOULD log it for debugging but MUST NOT parse or match against it |
| `pagination` | Object | Yes | Pagination info |
| `pagination.cursor` | String | No | Cursor for next page (`null` if last) |
| `pagination.cursorExpiresAt` | String | No | ISO 8601 expiry time for the cursor |
| `pagination.hasMore` | Boolean | Yes | Whether more results exist |

When no results match the query, the response is `"results": []` with `"hasMore": false`.

## Lookup Response (200)

Used by `GET /api/v1/posts/{id}`. See [Lookups](lookups.md#post-lookup).

```json
{
  "requestId": "req_lu789",
  "post": { ... }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requestId` | String | Yes | Unique request identifier |
| `post` | PostItem | Yes | The requested post. See [Posts](posts.md) |

## Navigation Response (200)

Used by `GET /api/v1/posts/{id}/replies` and `GET /api/v1/posts/{id}/quotes`. See [Lookups: Relationship Navigation](lookups.md#relationship-navigation).

```json
{
  "requestId": "req_rel456",
  "results": [ ... ],
  "pagination": {
    "cursor": "eyJvIjoyMH0=",
    "hasMore": true
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requestId` | String | Yes | Unique request identifier |
| `results` | Array | Yes | [PostItem](posts.md) results |
| `pagination` | Object | Yes | Pagination info (same structure as query responses above) |
| `pagination.cursor` | String | No | Cursor for next page (`null` if last) |
| `pagination.cursorExpiresAt` | String | No | ISO 8601 expiry time for the cursor |
| `pagination.hasMore` | Boolean | Yes | Whether more results exist |

There is no `algorithm` field; these endpoints use provider-default ordering.

Algorithmic relationship queries (`POST /api/v1/posts/{id}/replies/{algorithm}`) return the standard query response envelope above, including the `algorithm` field.
