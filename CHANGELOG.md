# Changelog — RFP: Enshrined Liquid Staking on Flare (eFLR)

All notable changes to this RFP. Versioned snapshots live in [`drafts/`](../drafts/); the push-ready set lives in [`github_upload/`](.).

## Draft 2 — 2026-09-07

Response to a structured gap review (ten points) plus post-publication edits. Adopted, adapted, or declined as follows.

- **§4.1 Decision matrix** added: seven criteria across Scopes A/B/C, two dated decision conditions (Q1 2027 primitive spec; Q3 2027 Coston2), cost column explicitly requested from the core team.
- **§5.2 position taken:** the P-chain primitive is the target, the FCC enclave a bridge with a sunset. **Appendix A** added: minimum specification for both routes (transaction type, proof pattern, epoch-bounded intents, image retirement constraint).
- **§6.3 terminology table** (node / entity / operator) and rule that the correlation penalty applies at entity level.
- **§6.5 identity-splitting attack** analysed. Reviewer's bond-weighted ramp *declined*: FIP.05's 15× delegation factor already prices Sybil entry in capital, and a bond-weighted ramp would slow the small honest operators the baseline exists for.
- **§6.6** yield-hopping: trailing window and clamp named as the instruments; both parameters.
- **§7.2 FIRE conflict** named: a fee to FIRE is a fee to the Foundation and gives it a stake in the pool's size. Fee destination now conditional on the custody condition in §8; burn otherwise. Enclave-loss arithmetic added (7M FLR/yr fee vs. pool-sized loss).
- **§8 transition and deadlock:** three staged custody stages with deposit-closing as enforcement; status-quo-wins rule; most-conservative-value rule; single-party emergency power limited to pausing deposits and new instructions. Reviewer's claim that FIP.16's committee is Foundation-appointed corrected: FIP.16 §4.5.1 specifies an election, gated behind a 50%-of-supply vote; the RFP reuses the election without the gate.
- **§9 FAssets** subsection: eFLR as agent collateral-pool asset, liquidity condition, haircut over liquidation lane. Reviewer's collateral factors (80/70) not adopted; no data.
- **§10.7 migration** rewritten: soft path does not move capital, hard path (Scope C) is what a rational Sceptre should prefer, force-conversion excluded in writing. **10.9** cost request, **10.10** FAssets acceptance added.
- **§11 monitoring protocol** (independent measurement, quarterly time series, three-miss review) and failure modes **11.9** inputs wrong / fail-closed, **11.10** regulatory, **11.11** enclave compromise.
- Post-publication edits folded in: companion line removed; proxy-with-kickbacks qualifier (§1, §2.3); fee marked as proposal (§7.2, README); disclosure corrected (occasional Spectra contact, duration interest); validator competition paragraph (§1, §6.6); §6.2 cross-reference fixed (7.6→7.2).
- Word count ~6,300 → ~9,200.

Commit: `Draft 2: decision matrix, settlement appendix, FIRE fee conflict, custody transition, FAssets, three failure modes`

## Draft 1 — 2026-09-06

First full draft, built from the FIP.17 sparring transcript and the "Ceiling on Owning" dispatch, fact-checked against FIP.05 / FIP.16 / Kiln P-chain docs.

Departures from the sparring spec, with reasons:

- **Settlement problem added (§5).** A C-chain contract cannot sign P-chain transactions; every Flare LST needs operator keys for the C→P leg. Enshrinement has to answer this (FCC-managed keys or a P-chain primitive on the FIP.16 Stage 3 path) or it is a Foundation-run private LST.
- **Three scopes instead of one (§4).** Router-only (Cosmos LSM pattern) is shippable without solving §5 and binds the existing pool; token requires custody.
- **No slashing on Flare's P-chain** → "slashing history" replaced by FIP.02 chill history; "insurance fund" replaced by a redemption reserve with a contract invariant; "principal-protected" dropped.
- **Cap denominator changed (§6.7)** from % of pool per node to % of *network* stake per entity, inclusive of non-router stake, so the router never pushes an entity across the governance cap.
- **Correlation rule added (§6.8)** — eight Sceptre nodes on one /24 showed the entity cap cannot see shared failure domains.
- **"Deflationary flywheel" dropped (§6.2).** Locking FLR changes float, not supply; the burn share of a 5% fee is immaterial against 3% inflation.
- **Fee split changed (§7.2):** reserve until invariant, then FIRE; no 50/50 burn/insurance.
- **Native PT/YT recommended against (§9);** Spectra rails exist. Conflict disclosed.
- **Governance (§8):** dual-veto kept as a parameter mechanism but named insufficient; custody question reframed with the four buckles (upgrade key, mandate governance, enclave image, input feeds); 2-of-3 custody minimum.
- **Ragequit redefined** as head-of-queue priority, not instant exit — instant exit against locked P-chain stake is not deliverable.
- **Ethereum objections (§3, §11.8)** included as the strongest external case against.

- **Token working name:** eFLR (enshrined FLR), replacing nFLR from the sparring draft; "native" is ambiguous on Flare, "enshrined" names the design lineage.

Commit: `Draft 1: enshrined liquid staking RFP — settlement gap, three scopes, router formula, custody envelope`
