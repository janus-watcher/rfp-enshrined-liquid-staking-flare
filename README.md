# RFP — Enshrined Liquid Staking on Flare (eFLR)

A request for proposal for a **protocol-owned liquid staking layer** on Flare: an algorithmic delegation router with per-entity caps the chain enforces, optionally a protocol-issued token (eFLR) under protocol custody, and the governance envelope that would have to bound it. Written against the September 2026 Sceptre case and the FIP.16 precedent of internalising an extraction the network could not police.

**Author:** Janus the Watcher · [@XRPWatcherJanus](https://x.com/XRPWatcherJanus)<br>
**Status:** Draft 4 (7 Sep 2026) — open for community review · see [CHANGELOG](CHANGELOG.md)

---

## TL;DR

Flare caps staking concentration by counting boxes: four nodes per entity, 300M FLR per node, enforced by a C-chain registry rather than the P-chain. The cap constrains the entity that runs validators and cannot see the pool that delegates to them. The largest pool holds 2.384bn FLR (11.08% of active stake) under no cap at all. When the permitted route started costing 20% of rewards (FIP.16) and the alternative was a second registration, the pool registered.

A percentage cap per entity is the first fix and belongs in FIP.02/FIP.05. It does not touch the second problem: capped on one route, capital moves to another, and a pool that may not run its own boxes delegates to strangers and buys them back with rebates. A proxy with kickbacks breaks no rule on the books and every intention behind them. This RFP asks whether Flare should stop regulating the pool and own it, the way FIP.16 stopped regulating MEV and designated a builder.

It sets out three scopes: **(A)** an enshrined delegation router that pools above a threshold must follow, enforced as a reward-eligibility rule (delegation off-target forfeits rewards) with a threshold schedule 11% → 8% → 5%; **(B)** a protocol-issued token, eFLR, under protocol custody; **(C)** both. Draft 3 recommends A as the product, in four independent deliverables (percentage cap; a non-binding allocation oracle first; A as reward filter, Songbird-tested; contract-originated P-chain delegation as its own FIP, not hung on the FIP.16 builder roadmap). B only behind that FIP or a reviewed FCC key set with the custody matrix filled in; otherwise, explicitly, not.

The router allocates by trust weight × net-yield weight, bounded by per-entity cap and remaining P-chain capacity, moved only from maturing delegations with a step limit and a published tracking error. Validator competition is not softened; it moves from relationships with pool operators to uptime, data quality and fee, and above the threshold it stops paying for marketing to stakers, on purpose, in line with FIP.16's shift to organic yield. Below the threshold the market is untouched. There is no slashing on Flare's P-chain, so there is no "insurance fund"; there is a redemption reserve with a contract invariant, an exit queue, a discount policy (nobody with admin access closes it; FIRE does not either), and a stress table. A protocol fee, proposed for discussion, goes to the reserve, then to FIRE only if the Foundation does not hold custody alone, and FIRE may not both seed the pool and receive the fee. Native PT/YT stripping is recommended against. The success metric for A and C is the share of pooled stake routed, not eFLR's market share. Failure modes lead with the strongest: an enshrined pool is itself the largest pool on the network, and its neutrality is whether one party holds image, upgrade key, feeds and parameter initiative at once.

## What's here

| File | Contents |
|---|---|
| [`RFP-Enshrined-Liquid-Staking-Flare.md`](RFP-Enshrined-Liquid-Staking-Flare.md) | Full RFP, readable on GitHub |
| [`RFP-Enshrined-Liquid-Staking-Flare.pdf`](RFP-Enshrined-Liquid-Staking-Flare.pdf) | PDF version for download / print |
| [`CHANGELOG.md`](CHANGELOG.md) | Version history |

## Structure

1. Executive Summary
2. The Problem — what the current rules measure; what FIP.16 changed; the proxy inevitability; the FIP.16 precedent (what is parallel, what is not); the do-nothing baseline
3. Precedents — Polkadot nomination pools, Cosmos LSM, Ethereum's road not taken, Flare's own market answer (LST kit, bond NFT)
4. Scope Options — router as reward filter / token / both; threshold schedule; decision matrix with dated conditions
5. The Settlement Problem — two chains, two delegations; FCC-managed keys vs a P-chain primitive (own FIP); mirror limit; exit queue, discount policy, griefing
6. Architecture — token accounting, capital sources, router formula with capacity and rebalance budget, eligibility, trust weight, net-yield weight and its endogeneity, cap, correlation (advisory), two yields
7. Economics — base rate, fee, who earns what, what the router does to the 20% floor and to staker marketing
8. Governance Envelope — immutable core, timelocked parameters, ragequit, custody matrix, staged transition, deadlock rule
9. Composability — why not to enshrine PT/YT; eFLR as FAssets collateral
10. Open Questions — eleven, ordered by design impact
11. Failure Modes — twelve, hardest first, with a monitoring protocol
12. What Ships First
Appendices — A settlement bridge spec · B numbers with dates · C liquidity under stress · D Songbird test plan · E threat model
13. Sources

## Contributing

Issues are open for discussion. Pull requests are welcome **as proposals** — reviewed and merged at the author's discretion to keep the document coherent. This is a curated RFP, not a wiki.

## License

[CC BY 4.0](LICENSE) — free to share and adapt with attribution.
