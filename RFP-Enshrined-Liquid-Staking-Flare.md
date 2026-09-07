# RFP — Enshrined Liquid Staking on Flare (eFLR)

**A design document for a protocol-owned liquid staking layer: what it would have to contain, which decisions it forces, and what would kill it.**

Author: Janus the Watcher · [@XRPWatcherJanus](https://x.com/XRPWatcherJanus)  
Status: Draft 1 — open for community review  
Date: September 2026  
Requires (reading): FIP.02, FIP.05, FIP.10, FIP.16  
Companions: [RFP — Native Options on Flare](https://github.com/janus-watcher/rfp-options-flare-native) (prices volatility) · RFP — Fixed-Term Lending on Flare (prices time and credit) · this one bounds the base asset both of them sit on.  

*Disclosure: the author holds FLR and sFLR, runs LP positions on Spectra with sFLR as underlying, and has an ongoing working relationship with Spectra. Sections 6 and 9 cut against that book. Weigh accordingly.*

*"FIP.17" is used below as a placeholder. The next FIP number is the Foundation's to assign. "eFLR" (enshrined FLR) is likewise a working name.*

---

## 1. Executive Summary

Flare limits staking concentration by counting boxes: four validator nodes per registered entity, 300 million FLR per node, a 15× delegation factor on a 1 million FLR self-bond. The limit is enforced by a C-chain registry, not by the P-chain, and it constrains the entity that runs validators. It cannot see the pool that delegates to them. The largest pool on the network holds 2.384 billion FLR, 11.08% of active stake, and is subject to no cap of any kind. The September 2026 Sceptre case showed what a rational operator does when the permitted route costs 20% of rewards and the alternative is a second registration: it registers.

A percentage cap per entity, floating with the network, is the first fix and belongs in FIP.02/FIP.05, not here. This document is about the second problem, the one a percentage cap does not touch. Capital that is capped on one route moves to another. A pool that may not run its own boxes delegates to strangers and buys them back with fee rebates. Governance can chase that for years. The alternative is to stop regulating the pool and to own it: a protocol-issued liquid staking token, eFLR, whose delegation is routed by a published formula across the validator set, with a per-entity cap the chain enforces because the chain is the delegator.

The precedent is FIP.16. Faced with MEV, an extraction the network could not police, Flare did not write a rule against extractors. It moved block building to a designated builder and routed the proceeds to FIRE. Private liquid staking is extraction of a different kind, of validator selection and of custody over pooled capital, and the same move is available.

This RFP lays out what an enshrined layer would need: the settlement problem (a C-chain contract cannot sign P-chain transactions, and that gap is where Sceptre's four keys live), the routing formula and its parameters, the fee and reserve design, the governance envelope, composability, and a migration path for the pools that exist today. It presents three scopes, not one: an enshrined router that existing pools must use, an enshrined token, or both. It then lists the ways it fails, starting with the strongest: an enshrined pool is itself the largest pool on the network, and its neutrality is exactly as good as the formula and the keys behind it.

The document is a request for proposal, not a specification. Numbers are proposals with reasons. The falsification tests in Section 11 carry dates.

---

## 2. The Problem: Capacity Is Not Independence

### 2.1 What the current rules measure

FIP.05 (September 2023) set the staking parameters that still govern: minimum self-bond 1 million FLR, delegation factor 15×, maximum 200 million FLR per node, up to four validators per infrastructure entity, uptime above 80% to earn rewards. The FIP.16 hard fork of 14 July 2026 raised the node maximum to 300 million. The entity ceiling is therefore 1.2 billion FLR.

Three things about that ceiling. It is absolute on a network whose active stake went from roughly 11 billion in March 2026 to 21.5 billion by September; a fixed number in a doubling system is a limit on growth wearing the costume of a limit on concentration. It is enforced by the entity registry on the C-chain, not by the P-chain, which FIP.05 says in plain words. And it measures the wrong layer. A node is a validator with its own stake; an entity is the registration that may run four of them; a pool is a contract that delegates to nodes it does or does not own. The rule constrains entities. The concentration sits in pools.

The one percentage that does exist, the 5% vote-power cap per validator above which delegators are diluted rather than refused, is slack: under a 300 million node cap no single node reaches 1.075 billion.

### 2.2 What FIP.16 changed

FIP.16 (April 2026) did four things that matter here. It cut inflation from 5% to 3%. It set a mandatory minimum 20% entity fee on everything staked or delegated to an entity. It raised the weight of P-chain stake to 5× C-chain delegation in the signing weight that governs FTSO, FDC and block-latency rewards, which pushes capital from liquid WFLR delegation into locked P-chain stake. And it created FIRE, the Flare Income Reinvestment Entity, with a mandate to capture fees and MEV and a roadmap that moves block building to a FIRE-designated builder in three stages.

Put together: the network made third-party delegation more expensive (20% floor), made P-chain staking more rewarding (5× weight), and left the enforcement of the entity limit to a registry that counts addresses. A liquid staking pool above 1.2 billion FLR now pays 20% on everything above the ceiling unless it finds a way around the ceiling. Sceptre found one. On the author's estimate the avoided fee was on the order of 30 million FLR a year, more than the pool's entire service-fee revenue.

### 2.3 The proxy inevitability

Assume governance fixes the count. Suppose FIP.02 is amended, the Management Group chills the second identity, and a binding 5% share of active stake per entity is written into the reward script. What does a 2.4 billion FLR pool do?

It delegates the excess to independent validators, and it negotiates. The FIP.16 floor guarantees those validators 20% of rewards; a side agreement returns part of it. The P-chain sees distinct NodeIDs. The registry sees distinct entities. The reward script sees a diversified delegation. The capital is as concentrated as before, and its concentration is now invisible.

This is not a hypothetical about Flare. It is the observed equilibrium of every proof-of-stake network that left liquid staking to the market: Lido reached roughly a third of Ethereum's stake through exactly this structure, a pool that owns no validators and selects all of them. A cap on the entity does not prevent it. A cap on the pool would, but the pool is a contract the Management Group cannot reach.

### 2.4 The FIP.16 precedent, stated precisely

The analogy that carries this document needs to be exact, because a loose version of it is wrong.

FIP.16 did not "ban MEV". It observed that MEV is a rational response to a protocol that leaves transaction ordering to whoever proposes the block, concluded that no rule against extractors would hold, and changed the protocol so that ordering is performed by a designated builder under a published mandate, with proceeds routed to FIRE. The extraction was internalized, not prohibited. The builder is initially the Foundation; Stage 2 moves the builder into Flare Confidential Compute; Stage 3 makes builder and proposer one protocol role.

The parallel to liquid staking is one-to-one. Validator selection by a private pool is rational under a protocol that leaves delegation to whoever holds the deposits. No rule against proxy delegation will hold. The alternative is a protocol that performs delegation itself, under a published formula, with the pool's custody moved from private keys to protocol keys. Whether that is worth its costs is the question this document exists to ask.

---

## 3. Precedents and Their Verdicts

Three networks have faced this and answered differently. The answers are the design space.

**Polkadot: nomination pools (2022).** Protocol-native staking pools, in runtime, with no liquid token. Anyone may create a pool; members pool stake under one nominator account. It solved the participation problem (minimum 1 DOT) and did nothing about concentration, because pool operators still choose validators. Lesson: enshrining the pool without enshrining selection changes custody, not concentration.

**Cosmos Hub: Liquid Staking Module (2023).** Not an enshrined token but an enshrined constraint on private ones. Liquid staking providers may hold at most 25% of all staked ATOM (governance-adjustable), and every validator must self-bond at 1:250 against the liquid stake delegated to it. Private LSTs remain; the protocol caps their aggregate and forces validator skin in the game. Lesson: a protocol can bound pools without owning them, if the chain can distinguish pooled delegation from direct delegation.

**Ethereum: the road not taken (2023–2025).** Enshrined LSTs were argued in earnest by Ethereum Foundation researchers, first as a two-tier staking model, then as rainbow staking (heavy and light services, protocol-enforced self-limits for pools). None has shipped. The objections are the ones this document must answer: a protocol that selects validators has moved the political question into protocol parameters; a protocol token competes with private ones but does not remove them; and enshrinement adds consensus-level surface for a problem that might be solved by capping issuance. Lesson: enshrinement is a minority path, and its advocates lost the argument on the largest network. Flare's case has to be made on Flare's specifics, not on Ethereum's.

The specifics are favourable in two ways. Flare's validator set is small (179 active), and its validators are already FTSO data providers whose performance the protocol measures every reward epoch on-chain. The information an algorithmic router needs is already being computed. And Flare has already crossed the line Ethereum would not: FIP.16 designated a builder. A network that has enshrined block building has fewer principled objections left to enshrining delegation.

---

## 4. Scope Options

The sparring that produced this document started from "the protocol issues eFLR". That is one of three scopes, and the cheapest one is not it.

**Scope A — Enshrined router, private tokens.** The protocol builds a delegation router: a contract that accepts delegation intent from any pool and executes it across the validator set by formula, with per-entity caps. Private LSTs continue to exist and to issue their own tokens, but above a threshold share of network stake they are required to delegate through the router (the Cosmos LSM pattern, applied to routing rather than to a global cap). Solves selection and concentration; leaves custody with the pools.

**Scope B — Enshrined token, protocol custody.** The protocol issues eFLR against deposits, holds the capital under protocol keys, and delegates by formula. Private tokens compete on the market. Solves custody and selection for the capital that migrates; does nothing for capital that stays in private pools unless Scope A is also present.

**Scope C — Both.** The protocol issues eFLR and requires every pool above the threshold to route through the same router eFLR uses. Private pools become front-ends on protocol rails; their remaining product is distribution, integration and yield strategy on top of the base rate.

The recommendation of this RFP is C, staged: A first, because it can ship without solving the settlement problem in Section 5 and immediately bounds the pool that exists today; B on the FIP.16 Stage 3 consensus change, because that change already moves where staked funds reside. A alone leaves the four-keys problem in place. B alone leaves Sceptre's 2.4 billion where it is.

---

## 5. The Settlement Problem

This is the section a reviewer from the core team will read first, and it is the one where the sparring draft was silent.

### 5.1 Two chains, two delegations

Flare inherits Avalanche's architecture: a P-chain that holds validator stake and delegation, and a C-chain that runs the EVM. Staking rewards for P-chain stake are computed off-chain by a public script and distributed by C-chain contracts through a mirror (`PChainStakeMirror`). FTSO delegation is a different mechanism entirely: WFLR on the C-chain, delegated by percentage to a data provider, no lock, rewards every 3.5 days. FIP.16 unified the weight both carry in the FSP protocols and set P-chain stake at 5×.

A liquid staking pool on Flare has to do both. Sceptre's contract holds deposits on the C-chain, delegates WFLR to FTSO providers, and moves a portion to the P-chain to stake against validators. The second step is the problem. A C-chain contract cannot construct or sign a P-chain transaction. Somebody has to export FLR from C to P, issue `addDelegator` or `addValidator`, and later import it back. That somebody holds keys. In Sceptre's case, four accounts with DEPOSIT and WITHDRAW roles, granted on 15 July 2026, that moved roughly 140 million FLR out of the pool into self-bonds with no timelock, no cap and no notice.

The four keys are not a Sceptre design flaw. They are the only way a C-chain pool can reach the P-chain today. Every liquid staking token on Flare has a version of them.

### 5.2 Three ways to close the gap

**(i) Protocol-managed keys in Flare Confidential Compute.** FIP.16 Stage 2 already plans to run the block builder inside FCC. The same enclave model can hold the P-chain keys of the eFLR pool: keys generated inside a TEE, never exported, executing a fixed policy (export, delegate per router output, import at maturity) against attested inputs. This is the author's earlier "Ulysses at the mast" proposal applied to staking rather than treasury. It is buildable on the FCC roadmap and it does not change consensus. Its weakness is the enclave image: whoever can replace it controls the pool, so the attestation and upgrade path for the image is the actual custody question.

**(ii) A P-chain primitive for contract-originated delegation.** The P-chain recognises a delegation whose owner is a C-chain contract address, with the mirror running in reverse: the contract emits intent, the P-chain executes it, the mirror proves execution. This is the clean answer. It is also a consensus change. FIP.16 Stage 3 says in terms that "the technical implementation of how exactly staking is handled, where the funds reside, and concrete staking processes will change in accordance with the consensus implementation changes." That sentence is the window. An enshrined pool designed now can ride that change; designed later it will have to be retrofitted.

**(iii) Restrict eFLR to C-chain delegation.** A pool that only delegates WFLR to FTSO providers needs no P-chain keys at all. It also earns at 1× weight against 5× for P-chain stake, and it does nothing for validator concentration, because it never touches validators. This is not a liquid staking token; it is a delegation wrapper, and it is mentioned so that it can be ruled out.

The RFP asks the core team one question above all others: is (ii) on the Stage 3 path, and if not, what would it cost to put it there? Everything in Section 7 assumes the answer to (i) or (ii) is yes.

### 5.3 Liquidity: the exit queue

P-chain delegation locks for a chosen period, minimum two weeks, and cannot be withdrawn early. A liquid token over locked capital needs either an instant-redemption reserve (Sceptre's model, and the reserve the four keys drew down) or an exit queue with a redemption delay, or both. The sparring draft assumed instant, zero-fee redemption during a governance timelock; that is not achievable against locked P-chain stake without a reserve large enough to honour it.

The proposal: a laddered delegation schedule (delegations staggered so that a fixed share matures every epoch), an exit queue served in order from maturing delegations, and an instant-redemption reserve whose minimum size is a contract invariant, not an administrator's discretion. The ragequit right in Section 8 is the right to enter the queue at the head, not the right to instant exit.

---

## 6. Architecture

### 6.1 Deposit and token

Users deposit FLR (or WFLR) and receive eFLR. Two accounting models exist. A rebasing token (balance grows) is simpler to reason about and breaks most DeFi integrations. A value-accruing token (balance fixed, exchange rate to FLR rises, sFLR's model) is what every lending market, AMM and yield-stripping protocol on Flare already integrates. eFLR should be value-accruing. This is not a small choice: it determines whether Spectra, Kinetic and Enosys can list eFLR on day one by cloning their sFLR configuration.

### 6.2 Capital sources

Two inflows. User deposits, which are the point. And, as an option the RFP puts on the table rather than assumes, protocol-owned capital: FIRE's mandate already ranks "rewards to FLR validators and stakers" as its second priority, and FIRE's captured FLR could be deposited into eFLR rather than distributed. The effect would be to seed the pool and to give FIRE a yield-bearing FLR position. Two cautions. FIRE's assets are mixed (stablecoins, FXRP, FLR), so only the FLR leg qualifies. And a pool part-owned by the Foundation's revenue entity is a pool whose neutrality will be questioned; Section 8 addresses whether the governance envelope can carry that.

The "deflationary flywheel" claim from the sparring draft is dropped. Locking FLR in a pool changes float, not supply. The fee in Section 7.6 burns a fraction of yield that is small against 3% inflation. eFLR is a supply sink in the FIP.16 sense (excluded from the inflatable base while locked) and nothing more, and the document should not say more.

### 6.3 The delegation router

The router is the mechanism that replaces Sceptre's validator analysts. Every reward epoch (3.5 days) it computes a target allocation of the pool's stake across eligible entities and issues the delegation instructions needed to move toward it, subject to lock schedules.

Target allocation per entity *i*:

```
A_i = W · (T_i · Y_i) / Σ_j (T_j · Y_j)        capped at C_max
```

where W is the pool's delegable stake, T is a trust weight, Y is a net-yield weight, and C_max is the per-entity cap. Surplus above any entity's cap redistributes to the next by the same weights. Delegation is per entity, summed over that entity's registered nodes, because the entity is what the registry knows and the node is what the P-chain knows; the router must reconcile both.

### 6.4 Eligibility (hard filters)

An entity is eligible if, over the evaluation window, it has met the FIP.05 uptime requirement (80%; the RFP proposes the router use a stricter 95%), has been rewarded by the FSP for FTSO data provision in every epoch of the window (which is what "good enough prices" means in FIP.05 terms and is already computed on-chain), holds no active chill under FIP.02, and has a self-bond meeting the P-chain minimum. There is no slashing on Flare's P-chain; "slashing history" in the sparring spec is replaced by chill history, which is the network's actual sanction record.

### 6.5 Trust weight T_i

New entities enter at T = 0.5 and rise linearly to T = 1.0 over 50 reward epochs (about six months) of continuous eligibility. The purpose is to make Sybil entry expensive in time, since a 1 million FLR self-bond does not make it expensive in capital. A window of 30 epochs (about 105 days) governs penalties: each epoch of ineligibility inside the window deducts 0.2 from T, floored at 0; eligibility restores T by 0.05 per clean epoch. An entity that goes dark for one epoch loses a fifth of its weight for four epochs and is back at full weight within a month. An entity that is chilled loses eligibility for the chill's duration and re-enters at T = 0.5. No lifetime exclusions; the router is not a court.

### 6.6 Net-yield weight Y_i

Flare's validators are data providers, and their reward rate varies with data quality. A router that ignores that pays bad data providers to exist. Y_i is the entity's realised net reward rate to delegators over the trailing 10 epochs, divided by the network median:

```
Y_i = (gross reward rate_i × (1 − fee_i)) / median_j(gross reward rate_j × (1 − fee_j))
```

Fee is the entity's declared fee, floored at 20% by FIP.16. An entity charging 35% competes against one charging 20% on net yield and loses delegation to it; an entity whose FTSO submissions earn more competes on gross and wins. Y is clamped to [0.5, 1.5] so that a single hot epoch cannot move the pool, and so that the router remains a baseline, not a momentum trader.

### 6.7 Cap C_max

The cap is the point of the whole design and the parameter most likely to be fought over. The sparring draft proposed 3% of the pool per node. That is the wrong denominator. A pool that is 40% of the network with a 3% per-node cap still hands 1.2% of the network to a node in one instruction; and a cap on the pool's share says nothing about the entity's total share once its other delegators are counted.

The proposal: C_max is expressed as a share of total active network stake, per entity, inclusive of the entity's stake from all sources as the reward script already sees it, and set so that the router never pushes an entity across the binding percentage cap this RFP assumes governance will adopt in FIP.02/FIP.05. If that cap is 5%, the router's C_max is 5% minus the entity's non-router stake. The router then does with code what the Management Group cannot do with a forum: it makes the largest delegator on the network incapable of concentrating.

### 6.8 Correlation

Eight of Sceptre's nodes sat in one /24 on one host. A cap per entity does not see that two entities share a rack. The router should carry a correlation penalty on top of C_max: entities whose nodes share an autonomous system, hosting provider or /24 with another eligible entity share one cap between them for the shared portion. The data exists (Catenalytica already attributes hosts); whether it can be made oracle-grade is an open question in Section 10.

---

## 7. Economics

### 7.1 The base rate

The pool's gross yield is the stake-weighted average of what its delegated entities earn, less their fees (20% floor), less the protocol fee below. At today's parameters an sFLR holder sees roughly 7–8% gross on the pool, of which Sceptre takes 10% and the network's 3% inflation absorbs most of the rest in real terms. The author's estimate of Sceptre's numbers: about 228 million FLR a year of pool rewards on 2.384 billion, about 23 million of which is service fee.

eFLR's advantage over that is not primarily the fee. It is that the router pays the 20% floor on everything and delegates within the entity ceiling by construction. A private pool that runs its own validators can beat eFLR on yield, exactly as Sceptre did, by taking the self-bond rewards for itself. That advantage is real and it is the advantage the network wants to remove. eFLR does not win on yield; it wins on not being the counterparty that can take the reserve.

### 7.2 Protocol fee

A 5% fee on pool rewards, versus Sceptre's 10% and typical LST fees of 10–20%. Two destinations, and the sparring draft's 50/50 split is replaced with something the numbers can carry.

The first destination is the instant-redemption reserve, until it reaches its invariant size (proposed: 5% of pool). Once the reserve is full, the fee goes entirely to the second destination: FIRE, under its existing mandate. The RFP does not propose a separate "insurance fund". Flare's P-chain does not slash, so there is no principal loss to insure against from validator conduct; the risks that remain (contract bug, enclave compromise, extended illiquidity) are not insurable out of 5% of yield, and a fund that claims to cover them is marketing. "Principal-protected" does not appear in this document.

### 7.3 Who earns what

The depositor earns the base rate net of 20% entity fees and 5% protocol fee, holds a token the counterparty cannot drain, and gives up the possibility of the above-market yield a self-validating pool can offer. The entity earns the 20% floor on router delegation, allocated by uptime, data quality and fee discipline rather than by relationship with a pool operator; small entities that perform get delegation they cannot market for today. The protocol earns a fee stream into FIRE and, more importantly, a validator set whose largest delegator is incapable of concentration by construction. The private pool operator loses the yield edge from self-bonding and the discretionary selection business, and keeps distribution, integrations and strategy layers.

### 7.4 What this does to the 20% floor

FIP.16's floor exists to make independent entities viable. A router that delegates by net yield puts every entity at the floor, because any fee above it loses delegation. That is the intended outcome and it should be said plainly: the router makes the floor the ceiling. If governance wants a market in entity fees above 20%, the Y weighting has to be softened. The RFP's position is that it should not be.

---

## 8. Governance Envelope

The sparring conversation's own objection: enshrining the pool moves the bottleneck from four private keys to whoever can change the contract. On Flare that is, today, the Foundation, through the governance timelock on every system contract. The FIRE precedent is instructive and not reassuring: FIRE is administered by the Foundation with a "negative governance" veto that requires 50% of the inflatable supply to vote, a threshold no vote has reached.

The envelope this RFP proposes has four parts, and the fourth is the one that matters.

**Immutable core.** Deposit, redemption, exit queue, the reserve invariant, and the cap formula are non-upgradable. If they are wrong, the pool is wound down and redeployed; it is not patched.

**Parameter set under timelock.** T ramp length, penalty size, Y clamp, uptime threshold, fee, reserve size, correlation rule: adjustable, with a 21-day on-chain timelock and public diff before effect.

**Ragequit during timelock.** Any pending parameter change opens a window in which eFLR holders may enter the exit queue at the head, ahead of ordinary redemptions, served from the reserve and maturing delegations. It is not instant exit; Section 5.3 explains why it cannot be. It is guaranteed priority.

**Custody outside the Foundation.** This is the part the sparring draft's dual-veto does not solve. A veto over parameters is worth little if the enclave image, the router's data feeds and the upgrade key are held by the same party. The four buckles from the author's FIRE essay apply verbatim: the upgrade key, the mandate governance, the enclave image, the input feeds. Each must have a documented holder, and for at least the enclave image and the upgrade key the holder must not be the Foundation alone. A 2-of-3 across Foundation, an elected entity committee (FIP.16's own joint-governance mechanism, Songbird-then-Flare election) and a time-delayed community key is the minimum. If the Foundation will not accept that, Scope A (router without protocol custody) is the honest fallback, and the four keys stay with the pools, bounded by the contract terms the earlier dispatch asked for.

---

## 9. Composability

eFLR is a base asset. Everything below it in the stack (fixed-term lending, options collateral, yield stripping) exists or is proposed elsewhere, and this section decides what the protocol builds and what it leaves.

The sparring draft proposed native PT/YT splitting in the protocol. This RFP recommends against it. Spectra already runs yield tokenisation on Flare against sFLR, with live pools across maturities, and the Fixed-Term Lending RFP already specifies PT-as-collateral on Spectra's rails. An enshrined splitter duplicates a live product, adds contract surface to the core, and forces the protocol to maintain a maturity calendar. What the protocol should do instead is make eFLR trivially strippable: value-accruing accounting, a clean exchange-rate oracle, no transfer hooks, no rebasing. Spectra then lists eFLR as it listed sFLR, and PT-eFLR becomes the collateral the lending RFP wants: a claim on protocol-custodied FLR with no operator key risk, which is the property that makes it institutional-grade rather than merely fixed-rate.

The author's conflict is on the table here: the recommendation favours a protocol the author works with. The counter-argument, that enshrined stripping removes a dependency on a third party, is real, and Section 10 keeps it open.

---

## 10. Open Questions

Ordered by how much the answer changes the design.

1. **Is contract-originated P-chain delegation on the FIP.16 Stage 3 path?** If yes, Scope B is a Stage 3 deliverable. If no, Scope B depends on FCC-managed keys and the enclave image becomes the custody question.
2. **Can the router's inputs be made oracle-grade?** Uptime and FTSO reward data are on-chain; host and AS attribution are not. The correlation rule (6.8) needs a source the protocol can verify or it stays advisory.
3. **Does the registry refuse a fifth node?** The earlier dispatch could not confirm it. If the registry enforces the count, Scope A can bind pools at the registry; if not, the mirror service is the enforcement point and phase 3 is a prerequisite.
4. **What threshold triggers mandatory routing under Scope A?** Cosmos chose 25% of all stake for the aggregate of liquid providers. Flare's largest pool is at 11%. A per-pool threshold of 5% of active stake is the number consistent with the entity cap; the RFP asks whether it should be lower.
5. **Should FIRE seed the pool?** Section 6.2 puts the option forward and names the neutrality cost. The answer depends on Section 8's custody outcome.
6. **Native stripping or Spectra rails?** Section 9 recommends rails. The dependency argument against it deserves a written answer from the Foundation, not from this author.
7. **What happens to sFLR?** A migration path is needed: a one-way sFLR→eFLR conversion at the exchange rate at snapshot, an incentive to use it, or nothing. Nothing is a legitimate answer under Scope C, where sFLR becomes a front-end over the router and its holders lose no yield.
8. **Songbird first?** Every consensus-touching FIP has shipped on Songbird first. The router (Scope A) can run on Songbird against SGB staking within a quarter of a decision; the token (Scope B) should not ship on Flare before it has run through at least two Songbird reward-epoch cycles with the exit queue under load.

---

## 11. Failure Modes

Hardest first.

**11.1 The enshrined pool is the largest pool.** If eFLR works it becomes 30–50% of active stake, and the largest delegator on the network is a contract whose parameters are set by governance and whose keys are held by whoever holds them. Every objection to Sceptre's 11% applies at three times the size. The design's defence is that the router cannot concentrate (6.7) and the keys cannot be used discretionarily (8). Both defences are exactly as strong as their implementation and no stronger. If the four buckles are not closed, this RFP has proposed a bigger Sceptre with a Foundation logo. Test: before mainnet, an independent review of the enclave image and key holders, published; if any single party can replace the image, do not ship Scope B.

**11.2 The router becomes the politics.** Every parameter in Section 6 is a decision about who gets delegation. Entities will lobby for the uptime threshold, the Y clamp, the correlation rule. The forum fight about Sceptre's identities becomes a permanent fight about T and Y. Mitigation: parameters under timelock with ragequit, so that a lost fight is an exit; and a rule that parameter changes take effect only at reward-epoch boundaries with a 21-day notice. This does not remove the politics. It gives it a schedule.

**11.3 The proxy problem survives.** A cap per entity inside the router does not stop a cartel of twenty entities from taking 60% of it. Section 6.8's correlation rule catches shared infrastructure; it does not catch shared ownership on separate infrastructure. The document should not claim otherwise. The honest statement: enshrinement removes the pool operator as the concentrating actor and leaves validator-level collusion where every proof-of-stake network leaves it, at the limit of what code can see. An identity-staking or later-established-breach slashing rule, whose buildability on Flare's P-chain is unresolved, is the next instrument, and it is not in this RFP.

**11.4 Nobody migrates.** sFLR is integrated in every venue on Flare, has a liquid market, and can offer above-router yield by self-validating. eFLR launches with a lower yield and no integrations. Capital does not move for safety it has not been made to feel. Under Scope B alone this is likely; the Cosmos LSM's 25% cap was never binding because private ATOM liquid staking never reached it. Under Scope C the question is moot for concentration (the pool routes through the protocol either way) and open for custody. Test: eighteen months after launch, eFLR below 10% of liquid-staked FLR under Scope B is a failed launch; under Scope C the metric is share of pooled stake routed, and below 80% means the threshold in 10.4 was set too high.

**11.5 The settlement gap is not closed.** If neither FCC-managed keys nor a P-chain primitive ships, Scope B is a private LST run by the Foundation, with the same four keys under a different name. In that case ship Scope A only, and say so.

**11.6 Liquidity crisis.** A run on eFLR during a market drawdown empties the reserve, the exit queue extends to the longest outstanding lock (up to a year), and eFLR trades at a discount on secondary markets. This is not a failure of the design; it is what a liquid token over locked stake does under stress, and every LST has done it. The failure is if the discount is used as an argument for administrator intervention in the reserve. The invariant in 5.3 exists to make that intervention impossible, and the document should expect the demand for it.

**11.7 The 20% floor collapses to the floor.** Section 7.4: the router makes 20% the ceiling. If the network's intention in FIP.16 was to allow entities to price above the floor, the router contradicts it. Governance should decide this on purpose.

**11.8 Ethereum was right.** The strongest external argument is that enshrinement was rejected on the network with the most research behind it, and that the rejection was on principle: a protocol that selects validators has taken a political role. Flare's answer is that it took that role in FIP.16 already and that its validator set is small enough to make an algorithmic router auditable. If the Foundation does not accept the first half of that answer, this RFP has no ground to stand on, and the right document is a FIP.02 amendment, not this one.

---

## 12. What Ships First

A percentage cap per entity in FIP.02/FIP.05, which needs no part of this document. Then Scope A on Songbird: the router, mandatory above a per-pool threshold, without protocol custody. Then, on the Stage 3 consensus change, Scope B with custody under the envelope in Section 8, or an explicit decision not to. The order matters because A bounds the pool that exists today and B does not exist until the settlement gap is closed.

The one thing this RFP asks the network not to do is to treat the Sceptre case as closed when the second identity is deregistered on 16 September. The identity was never the problem. The 2.4 billion is still there, still uncapped, and the next pool to reach the ceiling will do the arithmetic Sceptre did.

---

## 13. Sources

- FIP.05 — Update Services, Limits, and Rewards Required for Staking (accepted 8 Sep 2023): node/entity limits, delegation factor, uptime, "not enforced by the P-chain staking mechanism but by the mirroring service … until phase 3". proposals.flare.network/FIP/FIP_5.html
- FIP.16 — Restructure FLR Tokenomics for Long-Term Network Sustainability (accepted 24 Apr 2026): inflation 5%→3%, FIRE, single-builder MEV roadmap (Stages 1–3), 5× P-chain signing weight, node cap 200M→300M, 20% minimum entity fee, FIRE negative governance. proposals.flare.network/FIP/FIP_16.html
- FIP.02 — FTSO Management Group (chill / second-strike ban). proposals.flare.network/FIP/FIP_2.html
- Flare, Staking Phase 2 (Nov 2023); Flare Developer Hub, FLR staking documentation (5% vote-power reward cap per validator)
- Kiln, Flare (FLR) validator documentation: no slashing on P-chain; delegation lock 60–365 days; C-chain vs P-chain mechanics
- Forum proposal, Jon-Sn0w, forum.flare.network, 31 Aug 2026 (Rotko/Sceptre multiple identities; role grants 15 Jul 2026; withdrawal→self-bond sequences)
- Sceptre, "Sceptre response and remediation status", 4 Sep 2026; Sceptre, Monthly recap May 2026 (FIP.16 impact, own-validator plans)
- Flare Metrics, 5 Sep 2026: 21.51bn active stake, 179 validators, 8.6bn free delegation space, largest entity Bifrost 1.14bn
- Janus Dispatch, "The Ceiling on Owning" (6 Sep 2026): pool 2.384bn / 11.08%, fee-avoidance estimate ~30M FLR/yr, four-keys critical path, proposed contract bounds
- Janus Dispatch, "From Fee-Burn to Sovereign Capital" (19 Jul 2026): FIRE mandate ordering, autonomous-contract-in-FCC model, the four buckles
- Cosmos Hub, Liquid Staking Module (governance-approved 2023): 25% global liquid staking cap, validator bond factor 250
- Polkadot, Nomination Pools (Nov 2022): protocol-native pools, no liquid token
- Ethereum: V. Buterin, "Should Ethereum be okay with enshrinement?" (Sep 2023); "Possible futures of the Ethereum protocol, part 3: The Scourge" (Oct 2024); B. Monnot, "Unbundling staking: towards rainbow staking" (ethresear.ch, Feb 2024)
- Avalanche validator documentation: 2% minimum delegation fee, 5× delegation factor (for comparison with Flare's 20% / 15×)

*Figures marked as author's estimates are derived in "The Ceiling on Owning" and reproduced without recomputation.*

---

*Licensed CC BY 4.0. Pull requests are reviewed as proposals; this is a curated RFP, not a wiki.*
