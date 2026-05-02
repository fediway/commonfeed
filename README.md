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

CommonFeed is a JSON-over-HTTPS protocol for algorithmic feeds. Providers compute and rank content; consumers (servers, native clients, or web apps) query one or more providers and assemble the feed they show users. Requests carry no user identifiers.

## Status

v0.1 (prototype). Breaking changes are expected before v1.0.

Reference implementations:
- Provider: [fediway/feeds](https://github.com/fediway/feeds)
- Client: [fediway/fediway](https://github.com/fediway/fediway)

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

Request payloads carry no user identifiers, browsing history, or session data. Authentication is at the consumer level. Providers MUST NOT retain interest vectors beyond the request.

Anonymity properties scale with the consumer: multi-user consumers (e.g. fediverse servers) hide individual users in the consumer's traffic; single-user consumers (e.g. native clients) do not.

## Versioning

v0.x is exploratory: minor versions may include breaking changes. Each version is published at a stable path (`v0.1/`, `v0.2/`, …) so implementations can pin to a specific version. v1.0 will commit to backward-compatible evolution within the major version: new optional fields and endpoints only, no removals or semantic changes. Proposals are discussed in GitHub issues; accepted changes land in the next version directory.

## Relationship to FASP

CommonFeed sits alongside [FASP](https://github.com/mastodon/fediverse_auxiliary_service_provider_specifications) (Fediverse Auxiliary Service Provider). FASP specifies integration between fediverse instances and auxiliary services; CommonFeed specifies a query protocol for ranked content. The two address different layers and can be implemented together.

## Contributing

Feedback and proposals are welcome via GitHub issues.

## License

Specification text: [CC-BY-4.0](LICENSE).
