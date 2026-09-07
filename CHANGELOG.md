# Changelog — RFP: Enshrined Liquid Staking on Flare (eFLR)

All notable changes to this RFP. Versioned snapshots live in [`drafts/`](../drafts/); the push-ready set lives in [`github_upload/`](.).

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
