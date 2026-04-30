# Tag Results

Result items for `tags` resources. Each item represents a hashtag with usage statistics.

## Example

```json
{
  "name": "climate",
  "postCount": 142,
  "accountCount": 87,
  "history": [
    { "day": "1709251200", "uses": 30, "accounts": 12 },
    { "day": "1709164800", "uses": 25, "accounts": 10 }
  ],
  "score": 0.92
}
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Tag name (lowercase, no `#` prefix) |
| `postCount` | Integer | Yes | Total posts using this tag in the time window |
| `accountCount` | Integer | Yes | Unique authors using this tag in the time window |
| `history` | Array | No | Daily usage breakdown (most recent first) |
| `history[].day` | String | Yes | Unix timestamp of the start of the day (UTC), encoded as a string for compatibility with Mastodon's trends API |
| `history[].uses` | Integer | Yes | Posts using this tag on that day |
| `history[].accounts` | Integer | Yes | Unique authors on that day |
| `score` | Float | No | Relevance score in `[0, 1]`, normalized by the provider; higher means more relevant |
