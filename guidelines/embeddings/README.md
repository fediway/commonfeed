# Embedding Methods

Discovery's `embedding.methods` array names the methods a provider accepts. Each entry has a method `name` (e.g. `ema0.1`) and, when the method depends on an external embedding model, a `model`. Entries MAY also carry a `params` object for method-specific tunables that the consumer must apply when computing the vector. The method fixes how per-event embeddings are aggregated into the user vector; the model (when present) fixes the embedding space; `params` carries any additional values required to reproduce the vector deterministically.

The wire spec (vector layout, dimensionality, normalization, validation) lives in [Discovery](../../v0.1/discovery.md#embedding-capability) and [Queries](../../v0.1/queries.md#embedding). The per-method docs here cover what the wire spec deliberately leaves out: how the consumer turns user behavior into the vector that gets sent.

## Identifier conventions

Method identifiers SHOULD carry a version suffix (e.g. `ema0.1`). Aggregation rule changes that produce different vectors from the same behavior MUST bump the method version, since the identifier is the contract that lets two consumers reproduce the same vector.

A method MAY require a model (in which case the capability entry MUST declare one) or be self-contained (no `model` field on the entry). The method document declares which.

## What a method document covers

Each method document SHOULD cover:

- **Compatible models.** Which model identifiers this method is defined against, or that the method is self-contained.
- **Encoder.** What gets encoded per event (text only, text plus metadata, etc.), or that the method does not use a per-event encoder.
- **Aggregation rule.** The formula and any per-event-kind weights, with concrete numbers.
- **Cold start.** What a consumer sends before the user has produced any signal.
- **Negative signals.** How (or whether) hides, mutes, and similar are folded in.
- **Rotation and perturbation.** Concrete recipe for the consumer-side privacy guidance in the wire spec.

## Methods

- [`ema0.1`](ema0.1.md): EMA over per-post content embeddings, per-event-kind learning rates.

## Adding a new method

The spec does not gate registration. A provider that wants to advertise a new method publishes its specification (anywhere stable) and uses the identifier in its capability. Methods of broad interest can be contributed as files in this directory through a pull request.
