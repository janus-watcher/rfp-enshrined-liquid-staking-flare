# RFP — Enshrined Liquid Staking on Flare (eFLR)

A request for proposal for a **protocol-owned liquid staking layer** on Flare: an algorithmic delegation router with per-entity caps the chain enforces, optionally a protocol-issued token (eFLR) under protocol custody, and the governance envelope that would have to bound it. Written against the September 2026 Sceptre case and the FIP.16 precedent of internalising an extraction the network could not police.

**Author:** Janus the Watcher · [@XRPWatcherJanus](https://x.com/XRPWatcherJanus)<br>
**Status:** Draft 1 — open for community review · see [CHANGELOG](CHANGELOG.md)

---

## TL;DR

Flare caps staking concentration by counting boxes: four nodes per entity, 300M FLR per node, enforced by a C-chain registry rather than the P-chain. The cap constrains the entity that runs validators and cannot see the pool that delegates to them. The largest pool holds 2.384bn FLR (11.08% of active stake) under no cap at all. When the permitted route started costing 20% of rewards (FIP.16) and the alternative was a second registration, the pool registered.

A percentage cap per entity is the first fix and belongs in FIP.02/FIP.05. It does not touch the second problem: capped on one route, capital moves to another, and a pool that may not run its own boxes delegates to strangers and buys them back with rebates. A proxy with kickbacks breaks no rule on the books and every intention behind them. This RFP asks whether Flare should stop regulating the pool and own it, the way FIP.16 stopped regulating MEV and designated a builder.

It sets out three scopes: **(A)** an enshrined delegation router that pools above a threshold must use, without protocol custody; **(B)** a protocol-issued token, eFLR, under protocol custody; **(C)** both. It recommends A first (shippable on Songbird, binds the pool that exists today) and B on the FIP.16 Stage 3 consensus change, because a C-chain contract cannot sign P-chain transactions and that settlement gap is exactly where Sceptre's four keys live.

The router allocates by trust weight × net-yield weight, capped per entity as a share of *network* stake so the protocol's own pool is incapable of concentrating. There is no slashing on Flare's P-chain, so there is no "insurance fund"; there is a redemption reserve with a contract invariant and an exit queue. The 5% fee goes to the reserve, then to FIRE. Native PT/YT stripping is recommended against; Spectra rails already exist. Failure modes lead with the strongest: an enshrined pool is itself the largest pool on the network, and its neutrality is exactly as good as the formula and the keys behind it.

## What's here

| File | Contents |
|---|---|
| [`RFP-Enshrined-Liquid-Staking-Flare.md`](RFP-Enshrined-Liquid-Staking-Flare.md) | Full RFP, readable on GitHub |
| [`RFP-Enshrined-Liquid-Staking-Flare.pdf`](RFP-Enshrined-Liquid-Staking-Flare.pdf) | PDF version for download / print |
| [`CHANGELOG.md`](CHANGELOG.md) | Version history |

## Structure

1. Executive Summary
2. The Problem — what the current rules measure; what FIP.16 changed; the proxy inevitability; the FIP.16 precedent stated precisely
3. Precedents — Polkadot nomination pools, Cosmos LSM, Ethereum's road not taken
4. Scope Options — router only / token / both
5. The Settlement Problem — two chains, two delegations; FCC-managed keys vs a P-chain primitive; the exit queue
6. Architecture — token accounting, capital sources, the router formula, eligibility, trust weight, net-yield weight, cap, correlation
7. Economics — base rate, fee, who earns what, what the router does to the 20% floor
8. Governance Envelope — immutable core, timelocked parameters, ragequit, custody outside the Foundation (the four buckles)
9. Composability — why not to enshrine PT/YT
10. Open Questions — eight, ordered by design impact
11. Failure Modes — eight, hardest first
12. What Ships First
13. Sources

## Contributing

Issues are open for discussion. Pull requests are welcome **as proposals** — reviewed and merged at the author's discretion to keep the document coherent. This is a curated RFP, not a wiki.

## License

[CC BY 4.0](LICENSE) — free to share and adapt with attribution.
