<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset=".github/assets/logo-dark.svg" />
    <source media="(prefers-color-scheme: light)" srcset=".github/assets/logo-light.svg" />
    <img src=".github/assets/logo-light.svg" alt="CommonFeed" height="48" />
  </picture>
  <br />
  <br />
</p>

<p align="center">An open protocol for algorithmic feeds.</p>

CommonFeed is a JSON-over-HTTPS protocol with two roles: a provider computes and ranks content, and a consumer queries the provider and renders the results. The consumer mediates every request, so the provider never sees individual users. The protocol is forward-compatible by design. Unknown fields are ignored, so the spec can evolve without breaking deployments.

## Status

v0.1 (prototype). Breaking changes are expected before v1.0.

Reference implementation: [fediway/feeds](https://github.com/fediway/feeds).

## Specification

| Section | Description |
|---------|-------------|
| [Overview](v0.1/overview.md) | Protocol flow, transport, and privacy |
| [Discovery](v0.1/discovery.md) | Discovery document at `/.well-known/commonfeed` |
| [Registration](v0.1/registration.md) | Instance registration, verification, and key management |
| [Queries](v0.1/queries.md) | Query endpoint, filters, and pagination |
| [Lookups](v0.1/lookups.md) | Post lookup, replies, and quotes |
| [Responses](v0.1/responses.md) | Top-level response envelope |
| [Posts](v0.1/posts.md) | Post result items, media, link previews, and custom emojis |
| [Tags](v0.1/tags.md) | Tag result items |
| [Links](v0.1/links.md) | Link result items |
| [Errors](v0.1/errors.md) | Error codes and rate limit headers |

## Privacy

- Providers never see individual users. Consumers mediate every query.
- No browsing history, post identifiers, or user identifiers are sent to providers.
- When an algorithm accepts an aggregated interest vector, the vector does not reveal which specific content a user interacted with, and providers MUST NOT retain it beyond the request.
- Providers track usage at the consumer level only.

## Versioning

v0.x is exploratory: minor versions may include breaking changes. Each version is published at a stable path (`v0.1/`, `v0.2/`, …) so implementations can pin to a specific version. v1.0 will commit to backward-compatible evolution within the major version: new optional fields and endpoints only, no removals or semantic changes. Proposals are discussed in GitHub issues; accepted changes land in the next version directory.

## Relationship to FASP

CommonFeed sits alongside [FASP](https://github.com/mastodon/fediverse_auxiliary_service_provider_specifications) (Fediverse Auxiliary Service Provider). FASP specifies integration between fediverse instances and auxiliary services; CommonFeed specifies a query protocol for ranked content. The two address different layers and can be implemented together.

## Contributing

Feedback and proposals are welcome via GitHub issues.

## License

Specification text: [CC-BY-4.0](LICENSE).
