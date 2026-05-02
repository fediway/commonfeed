# Overview

A content provider exposes an HTTPS JSON API. Consumer instances register with the provider, then query for content. The provider returns ordered, displayable results.

All endpoints MUST use HTTPS. All request and response bodies are `application/json`.

## Protocol Flow

```
Instance                          Provider
   |                                 |
   |  GET /.well-known/commonfeed    |
   |-------------------------------->|   Discovery
   |  { endpoints, capabilities }    |
   |<--------------------------------|
   |                                 |
   |  POST /api/v1/register          |
   |-------------------------------->|   Registration
   |  { apiKey, verifyPath }         |
   |<--------------------------------|
   |                                 |
   |  Serve {verifyPath}             |
   |                                 |
   |  GET {domain}{verifyPath}       |
   |<--------------------------------|   Verification
   |  200 OK                         |
   |-------------------------------->|
   |                                 |
   |  POST {callbackUrl}             |   Webhook (if provided)
   |<--------------------------------|
   |                                 |
   |  GET /api/v1/register/status    |   Poll (alternative)
   |-------------------------------->|
   |  { status: "approved" }         |
   |<--------------------------------|
   |                                 |
   |  POST /api/v1/{resource}/{alg}  |
   |-------------------------------->|   Query
   |  { results[] }                  |
   |<--------------------------------|
   |                                 |
   |  GET /api/v1/posts/{id}         |
   |-------------------------------->|   Lookup
   |  { post }                       |
   |<--------------------------------|
   |                                 |
   |  GET /api/v1/posts/{id}/replies |
   |-------------------------------->|   Relationship
   |  { results[] }                  |
   |<--------------------------------|
```

After discovery, instances may also look up individual posts by provider-scoped ID and navigate relationships (replies, quotes). See [Lookups](lookups.md).

## Extensibility

Consumers MUST ignore unrecognized fields, object keys, and enumeration values. Consumers SHOULD process the parts of a response they understand and skip the parts they do not. This allows the protocol to evolve without breaking existing implementations.

## Privacy

- Request payloads carry no user identifiers, browsing history, or session data.
- Authentication is at the consumer level (one API key per consumer).
- Providers MUST NOT retain interest vectors (`embedding` field) beyond the lifetime of the request.
- Providers track usage at the consumer level only.

Anonymity properties scale with the consumer's user base. Multi-user consumers hide individual users in the consumer's traffic; single-user consumers reveal one user's query patterns to the provider through network metadata, though the wire payload remains identifier-free.

## Composition

Consumers MAY query multiple providers and merge results into a single feed. The merging strategy is a consumer concern. The `identifiers` field on result items (see [Posts](posts.md#protocol-identifier-formats)) enables deduplication of items returned by more than one provider. Consumers SHOULD cache responses to limit fan-out load on providers.

## Versioning

Two versions are independent:

- **Spec version**: the version of this specification (`0.1`, `0.2`, ...). Returned in the discovery document's `version` field. The spec evolves to add fields, refine rules, and accept new algorithms.
- **API path version** (`/api/vN`): the major version segment in URLs. Moves only on backward-incompatible wire format changes (removing or renaming fields, changing endpoint shapes).

The two evolve at different rates. v0.x of the spec stays at `/api/v1` while iterating. v1.0 will commit to backward-compatible evolution within `/api/v1`: new optional fields and endpoints only, no removals or semantic changes.
