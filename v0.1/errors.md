# Errors

All errors use standard HTTP status codes with a JSON body.

## Response

```json
{
  "requestId": "req_err456",
  "error": "error_code",
  "message": "Human-readable description",
  "param": "language"
}
```

All error responses include a `requestId` for debugging. The `param` field is included on `422` errors to indicate which parameter caused the failure. It is absent on other error types.

## Error Codes

| Status | Error Code | When |
|--------|-----------|------|
| 400 | `bad_request` | Malformed request body |
| 400 | `cursor_expired` | Pagination cursor has expired |
| 401 | `unauthorized` | Missing or invalid API key |
| 403 | `forbidden` | Valid API key but not authorized for this action |
| 404 | `not_found` | Unknown query type or resource. For [lookup endpoints](lookups.md): post does not exist or has been deleted |
| 422 | `invalid_params` | Valid JSON but invalid semantics (unsupported filter, out-of-range limit, etc.) |
| 429 | `rate_limited` | Rate limit exceeded |
| 500 | `internal_error` | Provider error |
| 503 | `unavailable` | Provider temporarily down |

When rate-limited (429) or unavailable (503), the provider MUST include a `Retry-After` header (in seconds).

## Rate Limit Headers

Providers SHOULD include rate limit headers on all responses:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 742
X-RateLimit-Reset: 1709312400
```

`X-RateLimit-Reset` is a Unix timestamp indicating when the limit resets.
