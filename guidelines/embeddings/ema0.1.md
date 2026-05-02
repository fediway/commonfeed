# Method: `ema0.1`

EMA over per-post content embeddings with per-event-kind learning rates.

## Compatible models

Any embedding model that accepts text input (text-only or multimodal). The vector this method produces inherits the dimensionality of the chosen model.

Example capability entry:

```json
{
  "name": "ema0.1",
  "model": "Qwen/Qwen3-Embedding-0.6B",
  "dims": 1024,
  "params": {
    "alpha": 0.05
  }
}
```

## Parameters

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `alpha` | Number | Yes | Base EMA learning rate. MUST be in `[0, 1]`. |

`alpha` is the provider-declared base decay rate. Each engagement event is scaled by a per-kind weight chosen by the consumer; the effective learning rate for a kind `k` is `α_eff[k] = clamp(alpha × weight[k], 0, 1)`. Setting `α_eff[k] = 0` causes events of kind `k` to have no effect on the user vector; setting `α_eff[k] = 1` causes them to replace prior state entirely.

Consumers MUST use the `alpha` value declared by the provider. Two providers advertising `ema0.1` with different `alpha` values will produce different decay characteristics from the same behavior; that is the provider's tuning.

## Engagement scaling

Engagement types and their relative weights are consumer-side concerns. A consumer applies whichever engagement primitives its host protocol surfaces, and SHOULD scale `alpha` per type so that stronger signals (typically those requiring more user effort or expressing more intent) drive larger updates than weaker ones.

`ema0.1` does not enumerate engagement types. Different consumers run on different protocols (ActivityPub, AT Protocol, Nostr, etc.) with different engagement primitives; a consumer's engagement vocabulary is its own to define.

The provider never observes individual engagement events. All events are applied consumer-side from the consumer's local state; the provider only sees the resulting unit vector.

### Example weight set

For a consumer exposing a Mastodon-style engagement vocabulary, a reasonable baseline:

| Kind | Recommended weight | Notes |
|------|---------------------|-------|
| `like` | 0.5 | Low effort, noisy positive signal. |
| `repost` | 1.0 | Public amplification. |
| `reply` | 1.5 | Writing effort and topical commentary. |
| `quote` | 2.0 | Public commentary with explicit reference to the source. |
| `bookmark` | 2.5 | Private intent ("I want to revisit this"); no social-performance overlay. |

With these weights and `alpha = 0.05`, effective alphas are:

| Kind | `α_eff` |
|------|---------|
| `like` | 0.025 |
| `repost` | 0.050 |
| `reply` | 0.075 |
| `quote` | 0.100 |
| `bookmark` | 0.125 |

Consumers SHOULD adapt these to their own engagement vocabulary and user base. The recommended values are guidance, not requirement.

## Encoder

The consumer encodes each engaged post into a unit vector using the model declared in the capability entry. When an engagement event represents a response to another post (e.g., reply, quote), the engaged post is the post being responded to, not the post the user authored. The text fed to the model is, in order, whatever of the following is present: author display name and handle, content warning, post title, post summary, post body, image alt text, hashtags. Fields that are absent are skipped. The exact template used by the reference implementation is at [`fediway/feeds/workers/orbit/templates/embedding.j2`](https://github.com/fediway/feeds/blob/main/workers/orbit/templates/embedding.j2).

The model's output is L2-normalized before aggregation.

### Quantization

Consumers MAY run quantized variants of the declared model (GGUF, AWQ, GPTQ, etc.) for efficient CPU inference. The model identifier in the capability refers to the base model; the quantization is a deployment choice on the consumer side. Quantized inference produces vectors that differ from full precision in low-order bits but remain semantically equivalent for similarity ranking.

### Example

A post with display name "Jane Doe", handle `jane@mastodon.social`, content warning "politics", body "Hello world", image alt texts "A sunset" and "A mountain", and tags "photography" and "nature" renders to:

```
Jane Doe (@jane@mastodon.social)

[CW: politics]

Hello world

[Image: A sunset]

[Image: A mountain]

#photography #nature
```

Sections are separated by a single blank line; absent fields are skipped along with their separator. The handle is prefixed with `@`; alt texts are wrapped in `[Image: ...]`; tags are prefixed with `#` and joined by single spaces. Leading and trailing whitespace are trimmed from the final string.

## Aggregation

For each engagement event in chronological order:

```
α_eff[k] = clamp(alpha · weight[k], 0, 1)
new = normalize(α_eff[k] · post_embedding + (1 − α_eff[k]) · current)
```

where `current` is the user's vector before the event, `post_embedding` is the unit-normalized post embedding, `k` is the event kind, `alpha` is the provider-declared base rate from `params.alpha`, and `weight[k]` is the consumer-chosen per-kind weight. The result is L2-normalized after every step. The user vector sent in the request is the value of `current` after applying all events.

## Cold start

A user with no prior engagement starts at the zero vector. Consumers SHOULD omit `embedding` in this state when the capability declares `embedding.required: false`. Behavior under `embedding.required: true` for cold-start users is not defined by `ema0.1` and is a candidate for a future revision.

## Negative signals

`ema0.1` does not aggregate hides, mutes, or similar negative signals into the user vector. These are out of scope for this method. A future method (`ema0.2` or sibling) may incorporate them.

## Rotation and perturbation

The wire spec recommends consumers rotate or perturb vectors periodically (see [Queries: Privacy](../../v0.1/queries.md#privacy)). For `ema0.1`, the recommended recipe is re-derivation from a fresh time window: drop the persisted `current` and rebuild the vector from engagements within the last N days, with N chosen by the consumer. Gaussian noise on the final vector (renormalized to unit length) is an acceptable alternative; the noise scale is consumer-determined.

## Reference implementations

- Provider (post encoding): [`fediway/feeds/workers/orbit`](https://github.com/fediway/feeds/tree/main/workers/orbit)
- Consumer (user vector aggregation): [`fediway/fediway/workers/orbit`](https://github.com/fediway/fediway/tree/main/workers/orbit)
