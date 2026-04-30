# Discovery

Providers publish a discovery document at `/.well-known/commonfeed`.

## Request

```
GET /.well-known/commonfeed
```

No authentication required.

## Response

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | Human-readable provider name |
| `domain` | String | Yes | Provider domain |
| `version` | String | Yes | Spec version |
| `baseUrl` | String | Yes | Base URL for all API endpoints |
| `capabilities` | Array | Yes | Available resource/algorithm combinations |
| `capabilities[].resource` | String | Yes | Resource type: `posts`, `tags`, `links` |
| `capabilities[].algorithm` | String | Yes | Algorithm name (e.g. `trending`, `search`) |
| `capabilities[].description` | String | Yes | Human-readable description |
| `capabilities[].params` | String[] | No | Accepted parameter names for this algorithm. Default `[]` |
| `capabilities[].requiredParams` | String[] | No | Parameters that MUST be provided. Subset of `params`. Default `[]` |
| `capabilities[].filters` | String[] | Yes | Supported filter names |
| `capabilities[].requiresOneOf` | String[] | No | At least one of these filters MUST be provided. Subset of `filters`. Requests missing all listed filters return `422`. Default `[]` |
| `capabilities[].multiValue` | String[] | No | Filters that accept arrays (multiple values, OR semantics). Filters not listed accept only a single value. Default `[]` |
| `capabilities[].embedding` | Object | No | Embedding support for this algorithm (see [Embedding Capability](#embedding-capability)). Absent if the algorithm does not accept embeddings |
| `lookups` | Object | No | Lookup capabilities. Absent if the provider does not support post lookups. See [Lookups](lookups.md) |
| `lookups.posts` | Object | No | Post lookup support. When present, `GET {baseUrl}/posts/{id}` is available |
| `lookups.posts.relationships` | String[] | No | Available relationship sub-resources (e.g. `"replies"`, `"quotes"`). When listed, `GET {baseUrl}/posts/{id}/{relationship}` is available. Default `[]` |
| `lookups.posts.algorithms` | String[] | No | Algorithm names that can be used with relationship queries (`POST {baseUrl}/posts/{id}/{relationship}/{algorithm}`). MUST be a subset of algorithm names declared in `capabilities` for the `posts` resource. Default `[]` |
| `maxResults` | Integer | Yes | Per-request hard cap on result count. Requests with `limit > maxResults` are silently clamped to `maxResults` |
| `rateLimits` | Object | No | Informational rate limits |
| `contact` | String | No | Operator contact |

Algorithmic query endpoints follow the pattern `POST {baseUrl}/{resource}/{algorithm}`. For example, `POST https://provider.example/api/v1/posts/trending`.

When `lookups` is present, lookup and relationship endpoints are also available. See [Lookups](lookups.md) for details.

<details>
<summary>Example</summary>

```json
{
  "name": "Example Content Provider",
  "domain": "provider.example",
  "version": "0.1",
  "baseUrl": "https://provider.example/api/v1",
  "capabilities": [
    {
      "resource": "posts",
      "algorithm": "trending",
      "description": "Trending content by engagement velocity",
      "params": ["window"],
      "filters": ["language", "protocol", "tag"],
      "multiValue": ["language", "tag"]
    },
    {
      "resource": "posts",
      "algorithm": "hot",
      "description": "Top content by engagement-weighted recency",
      "filters": ["language", "protocol", "tag", "link"],
      "requiresOneOf": ["tag", "link"],
      "multiValue": ["language", "tag"]
    },
    {
      "resource": "posts",
      "algorithm": "recommended",
      "description": "Personalized content ranked by interest similarity",
      "filters": ["language", "protocol"],
      "multiValue": ["language"],
      "embedding": {
        "required": true,
        "models": [
          { "name": "qwen3_256d", "dims": 256 }
        ]
      }
    },
    {
      "resource": "posts",
      "algorithm": "search",
      "description": "Full-text search across indexed content",
      "params": ["query"],
      "requiredParams": ["query"],
      "filters": ["language", "contentType", "protocol"],
      "multiValue": ["language", "contentType"]
    },
    {
      "resource": "tags",
      "algorithm": "trending",
      "description": "Trending hashtags by usage velocity",
      "params": ["window"],
      "filters": ["language"],
      "multiValue": ["language"]
    },
    {
      "resource": "links",
      "algorithm": "trending",
      "description": "Trending links by sharing velocity",
      "params": ["window"],
      "filters": ["language"],
      "multiValue": ["language"]
    }
  ],
  "lookups": {
    "posts": {
      "relationships": ["replies", "quotes"],
      "algorithms": ["trending", "hot"]
    }
  },
  "maxResults": 100,
  "rateLimits": {
    "requestsPerHour": 1000
  },
  "contact": "admin@provider.example"
}
```

</details>

## Embedding Capability

Capabilities MAY include an `embedding` field to advertise support for embedding-based ranking. When present, the algorithm accepts an interest vector as a query signal (see [Queries: Embedding](queries.md#embedding)). When absent, the algorithm does not accept embeddings.

The `embedding` object tells consumers which models the provider accepts, so they can compute a compatible vector before querying.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `embedding.required` | Boolean | Yes | Whether the embedding MUST be provided. If `true`, requests without an embedding return `422` |
| `embedding.models` | Array | Yes | Accepted embedding models. At least one |
| `embedding.models[].name` | String | Yes | Model identifier. Consumers use this to select and label their vectors |
| `embedding.models[].dims` | Integer | Yes | Expected vector dimensionality. Vectors with a different length are rejected |

A capability with `embedding.required: true` cannot return meaningful results without a vector; the embedding is the primary query signal. A capability with `embedding.required: false` uses the embedding as an optional ranking boost alongside other signals.

When a provider supports multiple models, consumers SHOULD prefer the first model in the array. Providers SHOULD order models by preference (recommended first).
