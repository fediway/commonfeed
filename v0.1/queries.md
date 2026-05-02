# Queries

Registered instances query for content.

## Request

```http
POST /api/v1/{resource}/{algorithm}
Content-Type: application/json
Authorization: Bearer cf_live_abc123xyz
```

The `{resource}` and `{algorithm}` must match a capability from the discovery document. If the combination is not recognized, the provider returns `404`.

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `params` | Object | No | Algorithm-specific parameters. Keys are parameter names, values depend on the parameter. Defaults to `{}`. See well-known params below |
| `filters` | Object | No | Key-value pairs to narrow results. Keys are filter names, values depend on the filter type. Defaults to `{}` |
| `embedding` | Object | Conditional | Interest vector for similarity ranking. Required when the capability declares `embedding.required: true` (see [Embedding](#embedding)). Absent or `null` otherwise |
| `limit` | Integer | No | Number of results to return. Range 1 to the provider's `maxResults` (see [Discovery](discovery.md)). Default 20. Values exceeding `maxResults` are silently clamped |
| `cursor` | String | No | Opaque cursor from a previous response for pagination. `null` or absent for the first page |

Providers MUST reject requests containing unsupported params with a `422` error. Requests missing a required param (declared in `requiredParams` in [Discovery](discovery.md)) return `422`.

### Well-Known Params

Providers and consumers SHOULD use the following well-known parameter names where applicable:

| Param | Type | Description |
|-------|------|-------------|
| `query` | String | Full-text search query |
| `window` | String | Time window: `hour`, `day`, or `week`. Default `day` when omitted |

Providers MAY define additional custom params beyond this list. Custom params MUST be declared in the capability's discovery entry.

```json
{
  "params": {
    "query": "climate policy EU"
  },
  "filters": {
    "language": ["en"]
  },
  "limit": 20,
  "cursor": null
}
```

## Filters

Filters narrow results. Each capability declares its supported filters in the discovery document. Providers MUST reject requests containing unsupported filters with a `422` error.

Most filters are optional. Some algorithms declare `requiresOneOf` in their capability entry (see [Discovery](discovery.md)); at least one of the listed filters must be provided in those cases. Requests missing all required filters return `422` with a descriptive error message.

Filters listed in a capability's `multiValue` array accept arrays (multiple values, OR semantics). Filters not listed in `multiValue` accept only a single value; providers MUST reject array values for single-value filters with `422`.

Providers and consumers SHOULD use the following well-known filter names where applicable, to ensure interoperability:

| Filter | Type | Description |
|--------|------|-------------|
| `language` | String / String[] | ISO 639-1 language codes |
| `contentType` | String / String[] | `post`, `article`, `reply` |
| `hasMedia` | Boolean | Filter to posts with media attachments |
| `protocol` | String / String[] | `activitypub`, `atproto`, `nostr` |
| `tag` | String / String[] | Hashtag names (lowercase, no `#` prefix). Returns content matching **any** of the provided tags (OR semantics). Accepts multiple values only if `"tag"` is listed in the capability's `multiValue` array (see [Discovery](discovery.md)); otherwise a single value only |
| `link` | String | URL of a link. Returns content that references this link |

Providers MAY define additional custom filters beyond this list. Custom filters MUST be declared in the capability's discovery entry.

## Embedding

Some algorithms accept or require an embedding: a dense vector representing the consumer's interest signal. The capability's `embedding` field in [Discovery](discovery.md#embedding-capability) describes whether embeddings are accepted, whether they are required, and which models are supported.

The embedding is a top-level request field, not a filter or parameter. It carries different semantics: filters narrow results, parameters tune algorithm behavior, but an embedding provides a query signal that determines result ranking.

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `embedding.vector` | Float[] | Yes | Interest vector. MUST be unit-normalized (L2 norm ≈ 1.0) |
| `embedding.method` | String | Yes | Method identifier. MUST match a `name` from the capability's `embedding.methods` array |
| `embedding.model` | String | Conditional | Model identifier. Required when the matched method entry declares a `model`; MUST be omitted otherwise. When present, MUST match the entry's `model` |

### Validation

Providers MUST validate the embedding and return `422 invalid_params` with a descriptive `param` and `message` if validation fails:

- **Missing required embedding**: if the capability declares `embedding.required: true` and the request omits `embedding`, return `422` with `param: "embedding"`.
- **Unknown method**: if `embedding.method` does not match any `name` in the capability's `embedding.methods` array, return `422` with `param: "embedding.method"`. The error message SHOULD list the supported method identifiers.
- **Model mismatch**: if the matched method entry declares a `model` and the request's `embedding.model` is missing or does not match, return `422` with `param: "embedding.model"`. If the matched method entry has no `model` and the request includes `embedding.model`, return `422` with `param: "embedding.model"`.
- **Wrong dimensions**: if `embedding.vector` length does not equal the matched entry's `dims`, return `422` with `param: "embedding.vector"`. The error message SHOULD include the actual and expected dimensions.
- **Zero vector**: if all elements of `embedding.vector` are zero, return `422` with `param: "embedding.vector"`. A zero vector has no direction and cannot produce meaningful similarity rankings.

### Privacy

The embedding is an aggregated interest signal, not browsing history. It does not contain identifiers for individual posts or authors. Consumers compute the vector locally by following the method declared in the capability (see [embedding methods](../guidelines/embeddings/)) and send only the resulting vector. Providers cannot reconstruct which specific content a user interacted with from the vector alone.

Consumers SHOULD rotate or perturb vectors periodically to limit long-term profiling. Providers MUST NOT store embeddings beyond the lifetime of the request.

<details>
<summary>Example: Recommended</summary>

Request:

```http
POST /api/v1/posts/recommended
Content-Type: application/json
Authorization: Bearer cf_live_abc123xyz
```

```json
{
  "embedding": {
    "vector": [0.023, -0.041, 0.019, "... 1024 floats total ..."],
    "method": "ema0.1",
    "model": "Qwen/Qwen3-Embedding-0.6B"
  },
  "filters": {
    "language": ["en", "de"]
  },
  "limit": 20
}
```

Response (see [Responses](responses.md) for envelope, [Posts](posts.md) for item schema):

```json
{
  "requestId": "req_rec789",
  "results": [
    {
      "url": "https://social.example/@dana/334455",
      "protocol": "activitypub",
      "identifiers": {
        "activitypub": "https://social.example/users/dana/statuses/334455"
      },
      "type": "post",
      "content": "<p>Fascinating paper on federated learning without central coordination.</p>",
      "text": "Fascinating paper on federated learning without central coordination.",
      "author": {
        "name": "Dana",
        "handle": "@dana@social.example",
        "url": "https://social.example/@dana",
        "identifiers": {
          "activitypub": "https://social.example/users/dana"
        }
      },
      "timestamp": "2026-03-24T08:30:00Z",
      "language": "en",
      "engagement": {
        "likes": 89,
        "reposts": 23,
        "replies": 7
      }
    }
  ],
  "algorithm": "commonfeed-recommended-v1",
  "pagination": {
    "cursor": "eyJvIjoyMH0=",
    "cursorExpiresAt": "2026-03-02T16:00:00Z",
    "hasMore": true
  }
}
```

</details>

## Pagination

- `limit`: 1 to the provider's `maxResults` (see [Discovery](discovery.md)). Default 20. Over-limit values are silently clamped
- `cursor`: opaque string from previous response, or `null` for first page
- Cursors are ephemeral. Providers SHOULD include `cursorExpiresAt` in the pagination object. Expired cursors return `400` with `error: "cursor_expired"`
