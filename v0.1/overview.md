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

- The provider never sees individual user identity. The instance mediates all queries.
- No browsing history, post identifiers, or user identifiers are sent to providers. Some algorithms accept an aggregated interest vector (see [Queries: Embedding](queries.md#embedding)), but the vector represents aggregate interest and cannot be reversed to identify specific content the user interacted with.
- Providers MUST NOT store interest vectors beyond the lifetime of the request.
- Providers track instance-level usage only.

## Versioning

Two versions are independent:

- **Spec version**: the version of this specification (`0.1`, `0.2`, ...). Returned in the discovery document's `version` field. The spec evolves to add fields, refine rules, and accept new algorithms.
- **API path version** (`/api/vN`): the major version segment in URLs. Moves only on backward-incompatible wire format changes (removing or renaming fields, changing endpoint shapes).

The two evolve at different rates. v0.x of the spec stays at `/api/v1` while iterating. v1.0 will commit to backward-compatible evolution within `/api/v1`: new optional fields and endpoints only, no removals or semantic changes.
