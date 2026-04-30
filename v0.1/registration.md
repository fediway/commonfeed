# Registration

Instances register to get an API key. The provider issues the key immediately, but it only becomes active after domain verification.

## Step 1: Request Registration

```http
POST /api/v1/register
Content-Type: application/json
```

```json
{
  "domain": "social.example",
  "name": "Example Social",
  "software": "mastodon",
  "softwareVersion": "4.3.0",
  "contact": "admin@social.example",
  "callbackUrl": "https://social.example/api/commonfeed/callback"
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `domain` | String | Yes | Instance domain |
| `name` | String | Yes | Instance display name |
| `software` | String | No | Software name |
| `softwareVersion` | String | No | Software version |
| `contact` | String | No | Operator contact |
| `callbackUrl` | String | No | Webhook URL for registration status notifications |

### Response (202)

```json
{
  "requestId": "req_reg001",
  "status": "pending",
  "apiKey": "cf_live_abc123xyz",
  "webhookSecret": "whsec_9f8e7d6c5b4a3210",
  "verifyPath": "/.well-known/commonfeed/cf_verify_x7k9m2p4",
  "expiresAt": "2026-03-02T15:30:00Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `requestId` | String | Unique request identifier |
| `status` | String | Always `"pending"` |
| `apiKey` | String | API key (inactive until verification) |
| `webhookSecret` | String | HMAC signing secret for webhooks. Returned only when `callbackUrl` was provided in the request. Distinct from `apiKey`; treat as confidential |
| `verifyPath` | String | Path the instance must serve for verification |
| `expiresAt` | String | ISO 8601 expiry time for the verification challenge |

The `apiKey` is issued immediately. Before verification it only works for `GET /api/v1/register/status`. After verification it works for all endpoints.

The `verifyPath` is the path the instance must serve on its domain. Verification expires after 1 hour. If it expires, the pending API key is revoked and the instance POSTs to `/api/v1/register` again to start a new flow.

## Step 2: Place Verification Path

The instance serves a `200` response at the `verifyPath` on its domain:

```http
GET https://social.example/.well-known/commonfeed/cf_verify_x7k9m2p4
```

The response body is ignored; the existence of the path is the proof. This follows the ACME challenge pattern (`/.well-known/acme-challenge/{token}`).

## Step 3: Provider Verifies

The provider fetches `https://{domain}{verifyPath}` and checks for a `200` response. This happens automatically in the background.

The provider MUST NOT follow HTTP redirects when fetching the verification path. Any 3xx response fails verification. This prevents domain hijacking via open redirects or subdomain takeovers on the target host.

The provider MUST resolve `{domain}` and reject verification if the resolved address is loopback (`127.0.0.0/8`, `::1`), link-local (`169.254.0.0/16`, `fe80::/10`), or RFC 1918 private (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`). DNS resolution MUST be re-checked at fetch time, not only at registration time, to prevent DNS rebinding.

## Step 4: Get Registration Status

The instance can check status at any time:

```http
GET /api/v1/register/status
Authorization: Bearer cf_live_abc123xyz
```

### Response Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requestId` | String | Yes | Unique request identifier |
| `status` | String | Yes | `pending`, `approved`, or `failed` |
| `rateLimits` | Object | No | Present when `status` is `approved` |
| `rateLimits.requestsPerHour` | Integer | No | Allowed requests per hour |
| `expiresAt` | String | No | Present when `status` is `pending`. ISO 8601 expiry time |
| `message` | String | No | Present when `status` is `failed`. Human-readable failure reason |

**Approved:**

```json
{
  "requestId": "req_abc123",
  "status": "approved",
  "rateLimits": {
    "requestsPerHour": 1000
  }
}
```

**Still Pending:**

```json
{
  "requestId": "req_def456",
  "status": "pending",
  "expiresAt": "2026-03-02T15:30:00Z"
}
```

**Verification Failed:**

```json
{
  "requestId": "req_ghi789",
  "status": "failed",
  "message": "Could not reach /.well-known/commonfeed/cf_verify_x7k9m2p4"
}
```

## Webhook Notification

If the instance provided a `callbackUrl`, the provider POSTs to it when verification completes.

`callbackUrl` MUST use `https://`. Providers MUST reject `callbackUrl` values that resolve to loopback (`127.0.0.0/8`, `::1`), link-local (`169.254.0.0/16`, `fe80::/10`), or RFC 1918 private network addresses (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`). Re-resolve at delivery time to mitigate DNS rebinding.

```http
POST https://social.example/api/commonfeed/callback
Content-Type: application/json
X-CommonFeed-Signature: sha256=HMAC_SHA256(webhookSecret, body)
```

```json
{
  "event": "registration.approved",
  "status": "approved",
  "domain": "social.example",
  "timestamp": "2026-03-02T14:35:00Z"
}
```

Or on failure:

```json
{
  "event": "registration.failed",
  "status": "failed",
  "domain": "social.example",
  "message": "Could not reach /.well-known/commonfeed/cf_verify_x7k9m2p4",
  "timestamp": "2026-03-02T14:35:00Z"
}
```

### Webhook Fields

| Field | Type | Description |
|-------|------|-------------|
| `event` | String | Event type: `registration.approved` or `registration.failed` |
| `status` | String | `approved` or `failed` |
| `domain` | String | Instance domain |
| `message` | String | Present on failure. Human-readable reason |
| `timestamp` | String | ISO 8601 event time |

The `X-CommonFeed-Signature` header contains an HMAC-SHA256 signature of the raw request body bytes (not normalized JSON) using the `webhookSecret` returned at registration as the signing key. Instances MUST verify this signature before processing the webhook to prevent spoofed callbacks.

Webhook delivery is best-effort. If the callback fails, the provider does not retry. The instance SHOULD fall back to polling `GET /api/v1/register/status`.

## Deregistration

```http
DELETE /api/v1/register
Authorization: Bearer cf_live_abc123xyz
```

The provider revokes the API key and removes the instance's registration. Returns `204 No Content` on success.

## Re-registration

An already-registered domain can register again at any time. The provider starts a new verification flow and issues a new pending API key. The previous key remains active until the new verification succeeds. At that point the old key is revoked and the new key becomes active. This ensures an instance is never locked out: failed re-registration attempts have no effect on the existing key.

This also serves as key recovery: if an instance loses its API key, it re-registers and verifies domain ownership to get a new one.

## Notes

The API key is a bearer token used for all authenticated requests. Instances MUST store it securely and MUST NOT expose it to end users.

The instance SHOULD remove the verification path after registration is approved.
