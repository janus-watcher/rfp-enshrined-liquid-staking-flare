# Changelog — RFP: Enshrined Liquid Staking on Flare (eFLR)

All notable changes to this RFP. Versioned snapshots live in [`drafts/`](../drafts/); the push-ready set lives in [`github_upload/`](.).

## Draft 3 — 2026-09-07

Response to a second review that found the strategic gap: the FIP.16 analogy carried less than Draft 1–2 claimed, and Scope A had no enforcement point.

- **Recommendation changed and the change made visible (§4, §12).** A is the product, in four independent deliverables; B only behind a delegation FIP or a reviewed FCC key set with the custody matrix filled in, otherwise explicitly not.
- **Stage 3 decoupled (§2.4, §5.2, OQ 1).** Stage 3 changes builder/proposer, not contract-originated delegation; the primitive is its own FIP. Analogy box: what is parallel (selection), what is not (settlement).
- **Scope A enforcement point named (§4):** reward-eligibility rule with tolerance ε against the published allocation; three possible edges listed, only the reward script is available without consensus change or custody. **Threshold schedule** 11% → 8% → 5% at 25-epoch steps.
- **Do-nothing baseline (§2.5)** quantified: ~15–19M FLR/yr in third-party fees on the excess, side-deal value ~9M.
- **Router (§6.3):** capacity term K_i (15× factor, 300M node cap); rebalance budget from maturities only; step limit; published tracking error. **T starts at 0.1** with one-fifth cap for 50 epochs (was 0.5). **Y endogeneity** named; rate-per-stake and cap as partial mitigations. **Correlation moved to advisory** (§6.8), out of C_max, until oracle-grade.
- **Mirror limit (§5.2):** three validators per P-chain address → vault architecture; Appendix A.2 corrected.
- **§6.9 Two yields:** P-chain/WFLR split as parameter, same entity weights for FTSO leg, contract claims and compounds; FTSO-selector power stated as hard as 11.1.
- **Liquidity (§5.3):** discount policy (no admin, no FIRE), griefing model, **Appendix C** stress table (5/10/20% over 1/30 days × 90/180/365-day ladders, 5% reserve).
- **Economics (§7):** net-to-depositor arithmetic 0.76g vs 0.78g; FIRE may not seed while receiving the fee; 5% not cemented pending Appendix C on Songbird data. Success metric for A/C moved to §1.
- **Governance (§8): custody matrix** (asset, signer, delay, key loss); committee = FIP.16 §4.5.1 seats without the 50% gate, veto only because the router pays them; community key 7-of-12, annual election, public ceremony, 14-day delay; FCC image reproducible build, CVE → pause not patch; wind-down as ladder run-off.
- **OQ 9 pool definition** (same privileged roles = same pool). **11.3** identity-staking announced as follow-up. **11.11** brand risk. **Appendix B** numbers with dates (Bifrost as live proof). **Appendix D** Songbird test plan. **Appendix E** threat model.
- §1 shortened by a third.
- Word count ~9,200 → ~13,000.

Commit: `Draft 3: A as product, reward-filter enforcement, Stage 3 decoupled, capacity-aware router, custody matrix, appendices B–E`

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
