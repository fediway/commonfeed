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
    "alpha": {
      "like": 0.05,
      "repost": 0.075,
      "reply": 0.15,
      "bookmark": 0.10
    }
  }
}
```

## Parameters

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `alpha` | Object | Yes | Per-event-kind EMA learning rates. Required keys: `like`, `repost`, `reply`, `bookmark`. Each value MUST be a number in `[0, 1]` |

`alpha[k]` is the weight given to a new event of kind `k` in the EMA update; `1 − alpha[k]` is the weight retained from prior state. Setting `alpha[k] = 0` causes events of that kind to have no effect on the user vector. Setting `alpha[k] = 1` causes events of that kind to replace prior state entirely.

The provider advertises `alpha` in the capability entry's `params` object. Consumers MUST use exactly the values declared by the provider. Two providers advertising `ema0.1` with different `alpha` values will produce different vectors from the same behavior.

## Encoder

The consumer encodes each engaged post into a unit vector using the model declared in the capability entry. The text fed to the model is, in order, whatever of the following is present: author display name and handle, content warning, post title, post summary, post body, image alt text, hashtags. Fields that are absent are skipped. The exact template used by the reference implementation is at [`fediway/feeds/workers/orbit/templates/embedding.j2`](https://github.com/fediway/feeds/blob/main/workers/orbit/templates/embedding.j2).

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
new = normalize(α[k] · post_embedding + (1 − α[k]) · current)
```

where `current` is the user's vector before the event, `post_embedding` is the unit-normalized post embedding, `k` is the event kind, and `α[k]` is the value of `params.alpha[k]`. The result is L2-normalized after every step. The user vector sent in the request is the value of `current` after applying all events.

## Cold start

A user with no prior engagement starts at the zero vector. Consumers SHOULD omit `embedding` in this state when the capability declares `embedding.required: false`. Behavior under `embedding.required: true` for cold-start users is not defined by `ema0.1` and is a candidate for a future revision.

## Negative signals

`ema0.1` does not aggregate hides, mutes, or similar negative signals into the user vector. These are out of scope for this method. A future method (`ema0.2` or sibling) may incorporate them.

## Rotation and perturbation

The wire spec recommends consumers rotate or perturb vectors periodically (see [Queries: Privacy](../../v0.1/queries.md#privacy)). For `ema0.1`, the recommended recipe is re-derivation from a fresh time window: drop the persisted `current` and rebuild the vector from engagements within the last N days, with N chosen by the consumer. Gaussian noise on the final vector (renormalized to unit length) is an acceptable alternative; the noise scale is consumer-determined.

## Reference implementations

- Provider (post encoding): [`fediway/feeds/workers/orbit`](https://github.com/fediway/feeds/tree/main/workers/orbit)
- Consumer (user vector aggregation): [`fediway/fediway/workers/orbit`](https://github.com/fediway/fediway/tree/main/workers/orbit)
