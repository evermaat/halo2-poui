# halo2-poui

A minimal patched fork of [zkonduit/halo2](https://github.com/zkonduit/halo2).

## Why this fork exists

`zkonduit/halo2` (the fork ezkl pins) requires nightly Rust because of a
single `#![feature(int_roundings)]` declaration in `halo2_proofs/src/lib.rs`.
The `int_roundings` feature was stabilized in **Rust 1.73 (October 2023)**,
so the gate is stale — the underlying methods (`div_ceil`, `next_multiple_of`,
etc.) compile on stable today.

The PoUI chain (`github.com/evermaat/poui-node`, a Substrate-based L1) needs
to link halo2 + snark-verifier into its node binary on stable Rust to verify
ezkl-produced ZK proofs in a host function. The unmodified zkonduit fork
cannot compile on stable, blocking that integration.

This fork removes the one stale `#![feature]` line. **No other changes.**
The fork tracks the exact ezkl-pinned commit (currently
`1dd2090741f006fd031a07da7f3c9dfce5e0015e`).

## Branches

- **`main`** — mirror of `zkonduit/halo2` upstream `main` (do not modify;
  used as the rebase base for `poui-stable-v1`).
- **`poui-stable-v1`** — the patched branch. Built from upstream commit
  `1dd20907` (the rev ezkl v23.0.5 transitively pins via its `[patch]`
  table) plus exactly one commit removing the nightly gate.

## Upstream contribution

A PR against `zkonduit/halo2` proposing the same change is open at
[zkonduit/halo2#27](https://github.com/zkonduit/halo2/pull/27). If upstream
merges, this fork can be retired.

## How to consume

In a downstream `Cargo.toml`:

```toml
[patch."https://github.com/zkonduit/halo2"]
halo2_proofs = { git = "https://github.com/evermaat/halo2-poui", branch = "poui-stable-v1" }
```

The `[patch]` table redirects all uses of `zkonduit/halo2` (direct and
transitive — e.g. via `snark-verifier`) to this fork.

## Maintenance

When ezkl bumps its `zkonduit/halo2` pin:
1. Fetch upstream into `main`.
2. Rebase `poui-stable-v1` onto the new commit.
3. If the patch still applies cleanly, push.
4. If a new nightly feature gate has been added, evaluate whether it is
   another stale gate (stabilized API) or a real nightly dependency. Stale
   gates: extend the patch. Real nightly: file an issue and stop.

## Provenance

Forked from `zkonduit/halo2` at commit `1dd20907`. Patch applied by Eric
(`evermaat87@gmail.com`) for the PoUI project, April 2026.
