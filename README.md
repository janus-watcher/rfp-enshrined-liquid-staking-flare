# RFP — Enshrined Disclosure on Flare

*A curve, not a router; eFLR only if custody can be shown*

A request for proposal for **stake ownership on Flare as a fact the chain publishes**, for **concentration priced in the reward script** by a curve on operator share instead of adjudicated in a forum, and for the protocol-owned liquid staking token (eFLR) that could sit behind both only if custody can be shown in a key matrix. Written against the September 2026 Sceptre case and the FIP.16 precedent of internalising an extraction the network could not police.

**Author:** Janus the Watcher · [@XRPWatcherJanus](https://x.com/XRPWatcherJanus)<br>
**Contributor:** Steven Hudspeth · [@hudspeth589](https://x.com/hudspeth589) (Drafts 4–6: disclosure-first, the disclosed/undisclosed axis, the curve in place of the router, shared-ABI correction, sFLR supply data)<br>
**Status:** Draft 6 (9 Sep 2026) — open for community review · see [CHANGELOG](CHANGELOG.md)

---

## TL;DR

Flare caps staking concentration by counting boxes: four nodes per entity, 300M FLR per node, enforced by a C-chain registry rather than the P-chain. The cap constrains the entity that runs validators and cannot see the pool that delegates to them. The largest pool holds 2.384bn FLR (11.08% of active stake) under no cap at all. When the permitted route started costing 20% of rewards (FIP.16) and the alternative was a second registration, the pool registered.

What failed in that case was not concentration but undisclosed concentration: a product sold as spreading stake, with most of it behind one operator across two identities, funded by privileged withdrawals from the pool it was custodying. A rule on size cannot tell that apart from a product that puts "one operator" on the label and lets depositors price it. Drafts 3–5 of this document proposed a rule on size. Draft 6 does not.

Three instruments, in order, none depending on the next. **Scope 0**, disclosure the chain publishes: entity identity, operator mapping, self-bond source, reward accounting, one script to check it; the one deliverable that would have surfaced Sceptre in July, and disclosure alone moved ~107M FLR out of the pool in eight days once it existed. **A percentage cap per entity** in FIP.02/FIP.05. **A curve**: rewards per unit of stake decline smoothly with an operator's share of the network, in the reward script, keyed to the mapping; Flare already runs a kinked version (the 5% vote-power cap per validator). Nobody decides where stake should sit; concentration is priced, not adjudicated, and an operator who wants it and says so can have it at the lower rate. The allocation formula survives as a **published oracle**, information only; the router of Drafts 3–5 is retired to Appendix G. **eFLR** stays where Draft 5 left it: only behind a delegation FIP or a reviewed key set with the custody matrix filled in; otherwise, explicitly, not.

The recommendation has moved three times in four days, each time toward less protocol. §4.1 says why. Failure modes lead with the ones that remain: the curve too flat, too steep, or gamed by identity splitting before the mapping exists; the proxy cartel on separate infrastructure, which no design here reaches; and, for eFLR, everything Draft 5 said.

## What's here

| File | Contents |
|---|---|
| [`RFP-Enshrined-Liquid-Staking-Flare.md`](RFP-Enshrined-Liquid-Staking-Flare.md) | Full RFP, readable on GitHub |
| [`RFP-Enshrined-Liquid-Staking-Flare.pdf`](RFP-Enshrined-Liquid-Staking-Flare.pdf) | PDF version for download / print |
| [`CHANGELOG.md`](CHANGELOG.md) | Version history |

## Structure

0. Definitions — eFLR, the curve, an operator, the oracle, the three numbers, the 20% floor, who pays
1. Executive Summary
2. The Problem — what the current rules measure; what FIP.16 changed; the proxy inevitability as a disclosure problem; the FIP.16 precedent (what is parallel, what is not); the do-nothing baseline
3. Precedents — Polkadot nomination pools, Cosmos LSM, Ethereum's road not taken, Flare's own market answer (LST kit, bond NFT)
4. What to Build — Scope 0 disclosure; percentage cap; the curve; the oracle as information; Scope B; why the recommendation moved; decision matrix
5. The Settlement Problem — two chains, two delegations; FCC-managed keys vs a P-chain primitive (own FIP); mirror limit; exit queue, discount policy, griefing
6. Architecture — token accounting, capital sources, the allocation oracle (formula, capacity, rebalance budget), eligibility, trust weight, net-yield weight, cap, correlation (advisory), two yields, the curve (§6.10)
7. Economics — base rate, fee, who earns what, what the curve does and does not do to entities
8. Governance Envelope — immutable core, timelocked parameters, ragequit, custody matrix, staged transition, deadlock rule
9. Composability — why not to enshrine PT/YT; eFLR as FAssets collateral
10. Open Questions — eleven, ordered by design impact
11. Failure Modes — thirteen, hardest first, with a monitoring protocol
12. What Ships First
Appendices — A settlement bridge spec · B numbers with dates (incl. sFLR supply series) · C liquidity under stress · D Songbird test plan · E threat model · F what FIP.16 says vs. what is inferred · G the retired router-as-reward-filter design
13. Sources

## Contributing

Issues are open for discussion. Pull requests are welcome **as proposals** — reviewed and merged at the author's discretion to keep the document coherent. This is a curated RFP, not a wiki.

## License

[CC BY 4.0](LICENSE) — free to share and adapt with attribution.
