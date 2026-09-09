# RFP — Enshrined Disclosure on Flare

## A curve, not a router; eFLR only if custody can be shown

**A design document for making stake ownership on Flare a fact the chain publishes, for pricing concentration in the reward script instead of adjudicating it in a forum, and for the protocol-owned liquid staking token that could sit behind both only if custody can be shown: what each would have to contain, which decisions they force, and what would kill them.**

Author: Janus the Watcher · [@XRPWatcherJanus](https://x.com/XRPWatcherJanus)  
Contributor: Steven Hudspeth · [@hudspeth589](https://x.com/hudspeth589) — Scope 0 (machine-readable disclosure), the disclosed/undisclosed axis, the curve in place of the router, the moat objection, the shared-ABI correction, the sFLR supply series; see Reviews  
Status: Draft 6 — open for community review  
Date: 9 September 2026 (Draft 1: 6 Sep; Drafts 2–4: 7 Sep; Draft 5: 8 Sep)  
Requires (reading): FIP.02, FIP.05, FIP.10, FIP.16  

*Disclosure: the author holds FLR and sFLR, runs LP positions on Spectra with sFLR as underlying, has had occasional contact with the Spectra team, and has an interest in duration markets on Flare existing. Section 6 cuts against the sFLR position; Section 9 runs with the duration interest. Weigh accordingly.*

*"FIP.17" is used below as a placeholder. The next FIP number is the Foundation's to assign. "eFLR" (enshrined FLR) is likewise a working name.*

---

## 0. Definitions, so the rest can be read

**What eFLR is.** The Scope B token: a protocol-issued, value-accruing claim on FLR held under protocol custody. Nothing else in this document issues a token, and the recommendation does not include it.

**What "the curve" is.** A change to the reward script, not to consensus: rewards per unit of stake attributed to an operator decline smoothly as that operator's share of active network stake rises above a starting point. Flare already has a version with one kink, the 5% vote-power cap per validator above which delegator rewards are diluted; the curve is that rule per operator and without the kink (§6.10).

**What an operator is.** The organisation that controls one or more registered entities, identified by the mapping Scope 0 publishes (§4). Not a node, not a registration.

**What "the oracle" is.** The allocation formula of Section 6, published every reward epoch as information: what a neutral delegator would do with the same inputs. It binds nothing. Draft 3 to Draft 5 proposed to enforce it as a reward filter above a threshold; that design is retired to Appendix G and the reasons are in Section 4.

**The three numbers.** Uptime over the evaluation window, the entity's realised FSP reward rate per unit of stake, and its declared fee, floored at 20% (§6.4–6.6). They are the oracle's inputs.

**What the 20% floor is a floor of.** The entity's share of rewards on anything staked or delegated to it, set network-wide by FIP.16 §5.2 "to prevent a race to the bottom".

**Who pays under the curve.** The delegators whose stake sits with a large operator, in the form of a lower rate on that stake. Not the operator, because Flare's P-chain does not slash and the only other place a sanction can land is the registry (FIP.02). The rate is the signal to the people who can move.

---

## 1. Executive Summary

Flare limits staking concentration by counting boxes: four validator nodes per registered entity, 300 million FLR per node, a 15× delegation factor on a 1 million FLR self-bond. The limit is enforced by a C-chain registry, not by the P-chain, and it constrains the entity that runs validators. It cannot see the pool that delegates to them, and it cannot see who owns the entity. The largest pool on the network holds 2.384 billion FLR, 11.08% of active stake. The September 2026 Sceptre case showed what a rational operator does when the permitted route costs 20% of rewards and the alternative is a second registration: it registers.

What failed in that case was not concentration. It was undisclosed concentration: a product sold as spreading stake, with most of it behind one operator across two identities, funded by privileged withdrawals from the pool it was custodying, under terms saying the operator never takes custody. A rule on size cannot tell that apart from a product that puts "one operator" on the label and lets depositors price the risk. This document's earlier drafts proposed a rule on size. This draft does not.

Three things, in order, none depending on the next. First, disclosure the chain publishes: entity identity, operator mapping, self-bond source, reward accounting, checkable by anyone with one script. That is the deliverable that would have caught Sceptre in July rather than September, it needs no consensus change, and the evidence for it is that disclosure alone moved about 107 million FLR out of the pool in eight days once it existed. Second, a percentage cap per entity in FIP.02/FIP.05, which needs no part of this document. Third, a curve: rewards per unit of stake decline smoothly with an operator's share of the network, in the reward script, keyed to the operator mapping. Nobody decides where stake ought to sit. Concentration is priced, not adjudicated, and an operator who wants it and says so can have it at the lower rate.

The delegation router of Drafts 3 to 5 is retired to an appendix. It required something to decide the right allocation and to enforce it as a cliff; a curve requires neither, and it removes the three failure modes the router carried: the protocol's allocator as the largest delegator, the allocator's parameters as permanent politics, and the allocator as the largest selector of data providers. The allocation formula survives as a published oracle, information only.

A protocol-owned token, eFLR, stays where Draft 5 left it: only behind a settlement primitive that does not exist or a reviewed key set with the custody matrix in Section 8 filled in, and otherwise, explicitly, not.

The recommendation has moved three times in four days, each time toward less protocol. That is a direction, and Section 4 says why.

---

## 2. The Problem: Capacity Is Not Independence

### 2.1 What the current rules measure

FIP.05 (September 2023) set the staking parameters that still govern: minimum self-bond 1 million FLR, delegation factor 15×, maximum 200 million FLR per node, up to four validators per infrastructure entity, uptime above 80% to earn rewards. The FIP.16 hard fork of 14 July 2026 raised the node maximum to 300 million. The entity ceiling is therefore 1.2 billion FLR.

Three things about that ceiling. It is absolute on a network whose active stake went from roughly 11 billion in March 2026 to 21.5 billion by September; a fixed number in a doubling system is a limit on growth wearing the costume of a limit on concentration. It is enforced by the entity registry on the C-chain, not by the P-chain, which FIP.05 says in plain words. And it measures the wrong layer. A node is a validator with its own stake; an entity is the registration that may run four of them; a pool is a contract that delegates to nodes it does or does not own. The rule constrains entities. The concentration sits in pools.

The one percentage that does exist, the 5% vote-power cap per validator above which delegators are diluted rather than refused, is slack: under a 300 million node cap no single node reaches 1.075 billion.

### 2.2 What FIP.16 changed

FIP.16 (April 2026) did four things that matter here. It cut inflation from 5% to 3%. It set a mandatory minimum 20% entity fee on everything staked or delegated to an entity. It raised the weight of P-chain stake to 5× C-chain delegation in the signing weight that governs FTSO, FDC and block-latency rewards, which pushes capital from liquid WFLR delegation into locked P-chain stake. And it created FIRE, the Flare Income Reinvestment Entity, with a mandate to capture fees and MEV and a roadmap that moves block building to a FIRE-designated builder in three stages.

Put together: the network made third-party delegation more expensive (20% floor), made P-chain staking more rewarding (5× weight), and left the enforcement of the entity limit to a registry that counts addresses. A liquid staking pool above 1.2 billion FLR now pays 20% on everything above the ceiling unless it finds a way around the ceiling. Sceptre found one. On the author's estimate the avoided fee was on the order of 30 million FLR a year, more than the pool's entire service-fee revenue.

One objection to this document, from operators who build on Flare, is that Sceptre was not a product failure but a failure of the people holding the keys, and that no formula fixes a person. The first half is true and the second half is beside the point. Whether an operator is careful or careless decides how ugly the incident looks, not whether it happens. A fully funded operator at 11% of stake would not have needed pool money for self-bonds; it would have built the same seven boxes with its own capital and registered the same second identity, because a 20% floor over a box ceiling produces that route for anyone above the ceiling, and the arithmetic does not ask who is doing it. The incident would have been cleaner. The concentration would have been the same. This document is about the parameters that make the route rational, not about the people who took it.

### 2.3 The proxy inevitability, and what it is a problem of

Assume governance fixes the count. Suppose FIP.02 is amended, the Management Group chills the second identity, and a binding 5% share of active stake per entity is written into the reward script. What does a 2.4 billion FLR pool do?

It delegates the excess to independent validators, and it negotiates. The FIP.16 floor guarantees those validators 20% of rewards; a side agreement returns part of it. The P-chain sees distinct NodeIDs. The registry sees distinct entities. The reward script sees a diversified delegation. The capital is as concentrated as before, and its concentration is now invisible. Nothing in this is illegal, and nothing in it is what FIP.05 or FIP.16 meant.

Earlier drafts treated this as a second problem, distinct from custody, and proposed to solve it by having the protocol perform the selection. This draft reads it differently, and an operator reviewer is the reason. A pool that spreads two billion across twenty entities and chooses them is doing something depositors may or may not want, and on a network with a 15× delegation factor, where no entity can take more than fifteen times its own bond, the harm of one pool's choice is bounded. What is not bounded is the harm of not knowing: which entities are one operator, which self-bonds came from which pool, where the rewards went. Every element of the Sceptre case that mattered was a fact nobody could see. The proxy problem is a disclosure problem with an economics problem behind it, and the two instruments in Section 4 are aimed at those. A side agreement that stays off-chain stays invisible under any design in this document, and the document should not claim otherwise.

This is not a hypothetical about Flare. It is the observed equilibrium of every proof-of-stake network that left liquid staking to the market: Lido reached roughly a third of Ethereum's stake through exactly this structure, a pool that owns no validators and selects all of them, and Ethereum's answer has been to argue about it for three years.

### 2.4 The FIP.16 precedent, stated precisely

The analogy that carries this document needs to be exact, because a loose version of it is wrong.

FIP.16 did not "ban MEV". It observed that MEV is a rational response to a protocol that leaves transaction ordering to whoever proposes the block, concluded that no rule against extractors would hold, and changed the protocol so that ordering is performed by a designated builder under a published mandate, with proceeds routed to FIRE. The extraction was internalized, not prohibited. The builder is initially the Foundation; Stage 2 moves the builder into Flare Confidential Compute; Stage 3 makes builder and proposer one protocol role.

The parallel holds for selection and breaks for settlement, and the document should not let the reader blur them.

| | MEV (FIP.16) | Liquid staking (this RFP) |
|---|---|---|
| Extraction | Transaction ordering by whoever proposes | Validator selection and custody by whoever holds deposits |
| Why rules fail | Extractors are rational, ordering is invisible | Proxies are rational, side-deals are invisible |
| Protocol answer | Designated builder under a mandate | A fact the chain publishes and a price in the reward script; no designated anything |
| What it needed | Consensus change to block validation | For disclosure and the curve: a reward-script change, which is how the 5% cap already works. For custody: a P-chain primitive that does not exist |
| Where Stage 3 helps | Directly; it is the Stage 3 deliverable | Not directly. Stage 3 changes builder and proposer roles and says stake "will continue to count towards the vote power"; it reopens staking mechanics, it does not specify contract-originated delegation |

Extraction by a private pool is rational under a protocol that leaves it unpriced, and no rule against it will hold; that half of the analogy is exact. The difference is that FIP.16 answered MEV by designating an actor, and this document answers concentration by pricing it, which is the smaller move. The custody half is a request, not a precedent.

### 2.5 The do-nothing baseline

Suppose governance adopts the percentage cap and the FIP.02 sanction and nothing else. Twelve months out, the arithmetic is the one Sceptre already did. The pool is 2.384 billion; 1.2 billion may sit under one entity; 1.18 billion must be delegated elsewhere. At the current reward rate on the delegated portion (Appendix B), that 1.18 billion earns on the order of 75 million FLR a year for the pool and 19 million of it goes to third-party entities as the 20% floor. A side agreement that returns half of that is worth about 9 million FLR a year to the pool and costs the entity nothing it would otherwise have had. There is no rule against it, no chain data that shows it, and no reason a rational operator would not do it. The baseline is not "the cap holds"; it is "the cap holds on paper and 1.2 billion is delegated on terms nobody can see". Any proposal in this document has to beat that baseline, not a strawman in which capital sits still. Scope 0 beats it by making the side agreement's precondition, that nobody can see who owns what, false. The curve beats it by making the operator's own-node share, the thing the side agreement protects, cost more the larger it is.

---

## 3. Precedents and Their Verdicts

Three networks have faced this and answered differently. The answers are the design space.

**Polkadot: nomination pools (2022).** Protocol-native staking pools, in runtime, with no liquid token. Anyone may create a pool; members pool stake under one nominator account. It solved the participation problem (minimum 1 DOT) and did nothing about concentration, because pool operators still choose validators. Lesson: enshrining the pool without enshrining selection changes custody, not concentration.

**Cosmos Hub: Liquid Staking Module (2023).** Not an enshrined token but an enshrined constraint on private ones. Liquid staking providers may hold at most 25% of all staked ATOM (governance-adjustable), and every validator must self-bond at 1:250 against the liquid stake delegated to it. Private LSTs remain; the protocol caps their aggregate and forces validator skin in the game. Lesson: a protocol can bound pools without owning them, if the chain can distinguish pooled delegation from direct delegation.

**Ethereum: the road not taken (2023–2025).** Enshrined LSTs were argued in earnest by Ethereum Foundation researchers, first as a two-tier staking model, then as rainbow staking (heavy and light services, protocol-enforced self-limits for pools). None has shipped. The objections are the ones this document must answer: a protocol that selects validators has moved the political question into protocol parameters; a protocol token competes with private ones but does not remove them; and enshrinement adds consensus-level surface for a problem that might be solved by capping issuance. Lesson: enshrinement is a minority path, and its advocates lost the argument on the largest network. What Ethereum's research did leave behind is the idea this document now uses: penalties or issuance that scale with share of stake, so that concentration is priced rather than adjudicated. Flare's case has to be made on Flare's specifics, and Flare's specific is that it already runs a kinked version of that price, the 5% cap.

**Flare: the market's own answer (2026).** Two Flare operators, Steven Hudspeth and Jon, have built the two halves of this document's problem without the Foundation, and the RFP would be dishonest not to put them in the precedent list. A bond NFT: capital picks its operator in public, on an immutable contract with no upgrade key and a multisig on the funds, and what it funds is self-bond, the capacity the router in Section 6 can only wait for. And an LST kit: an immutable token, modules behind a timelock, a single emergency power that is a pause and expires, keys bounded so that a theft is a leak rather than a drain, one instance per provider with its own keys, and operators auditing each other's contracts before deployment. Contract addresses and terms are to be added when the authors supply them.

On custody the kit is the answer Section 5 asks for, and the "Ceiling" dispatch asked Sceptre for, and it exists today. On selection and on concentration it is not yet an answer, though the obvious objection, that Kinetic will not list twenty tokens and Spectra will not build twenty curves, is weaker than it looks. Every instance is deployed from one frozen template with one ABI, so an integrator writes one adapter and a registry of instances deployed from the verified bytecode enumerates the set, which is how ERC-4626 vaults are integrated today. That is correct, and it removes the engineering half of the objection; it also makes the registry itself an instance of Scope 0. What it does not remove, and the authors say so, is the other half: a shared ABI does not set a collateral factor or make a thin market deep, and liquidity and risk parameters are per instance. So the question is not twenty tokens versus one. It is whether onboarding can be made cheap enough that twenty thin markets behave like one deep one, and the answer to that is not in this document or yet in theirs. Until it is, the two outcomes stand: instances stay small and thin while capital stays where depth is, or one consolidates and becomes sFLR with a better custody model and the same selection problem. And a bond NFT is capital choosing its operator in public, which is transparent and is the mechanism that built Lido; visibility changes who can see the choice, not who makes it. One more thing the kit does, and it is the thing that moved this document: it puts the concentration on the label. One token, one provider, the operator risk priced in the discount between provider tokens. That is concentration as an informed choice, and any rule that penalises it for succeeding has measured the wrong thing. Lesson: the market on Flare has bounded custody and has shown what disclosed concentration looks like. What it cannot do is publish the facts about pools that do not want them published, and that is what Section 4 is for.

The specifics are favourable in two ways. Flare's validator set is small (179 active), and its validators are already FTSO data providers whose performance the protocol measures every reward epoch on-chain. The information an algorithmic router needs is already being computed. And Flare has already crossed the line Ethereum would not: FIP.16 designated a builder. A network that has enshrined block building has fewer principled objections left to enshrining delegation.

---

## 4. What to Build

Three instruments and one token. The instruments do not depend on each other and are listed in the order they should ship.

**Scope 0 — Disclosure the chain publishes.** Entity identity; the mapping from registered identities to operators; the source of every self-bond, meaning the address it was funded from and whether that address holds privileged roles on a pool contract; the reward-owner fields per node; reward accounting per identity per epoch. Published on-chain in a form that one script can check and every script gets the same answer from. This is what the forum proposal of 31 August reconstructed by hand from registration transactions over several weeks; it is what FIP.02's "evidence" clause leaves undefined; and it is the one deliverable in this document that would have surfaced the Sceptre structure in July. It needs no router, no threshold and no consensus change. It is also the precondition for every other instrument here, because both the curve and the retired cliff are gamed by identity splitting until the operator mapping exists. The case for sequencing it first is not theoretical: the sFLR pool was flat for three weeks before 31 August and fell by about 107 million FLR in the eight days after the evidence went up, with a quarter moving to a competing token and the rest leaving pooled staking (Appendix B). No mechanism did that.

**A percentage cap per entity in FIP.02/FIP.05.** A share of active stake, floating with the network, computed by the reward script that already computes each node's share. Not part of this document; listed so the order is complete.

**The curve.** Rewards per unit of stake attributed to an operator decline smoothly as the operator's share of active network stake rises above a starting point, in the reward script, keyed to the Scope 0 mapping. No published target, no allocator, no epoch decision, nobody deciding where capital ought to sit. Concentration becomes progressively more expensive and capital diffuses on its own, or does not, and either way the choice is the depositors' at a price they can see. Flare has this already in kinked form: the 5% vote-power cap per validator dilutes delegator rewards above the line every epoch. The curve is that rule per operator instead of per validator, smooth instead of stepped, with its parameters set under FIP.11's methodology. Section 6.10 gives a shape. It reaches the pool the Management Group cannot reach, because it is a price and not a verdict, and it does not need a room the defendant cannot enter.

**The allocation oracle.** The formula of Section 6, published every reward epoch and binding nothing: what a neutral delegator would do with public inputs. It gives depositors a benchmark and gives the curve's parameters a reference. Drafts 3 to 5 proposed to enforce it as a reward filter above a threshold; that design, with its threshold schedule and its tolerance ε, is in Appendix G with the reasons it was retired.

**Scope B — Enshrined token, protocol custody.** Unchanged from Draft 5: the protocol issues eFLR against deposits, holds the capital under protocol keys, and delegates by the oracle. Only behind a contract-originated P-chain delegation FIP (§5.2 ii) or a reviewed FCC key set (§5.2 i) with the custody matrix in Section 8 filled in. If neither exists, B does not, and that is a legitimate outcome of this RFP.

### 4.1 Why the recommendation moved

Draft 1 recommended an enshrined token with a router behind it. Draft 3 recommended the router alone, as a reward-eligibility cliff above a threshold, because the token's custody could not be shown. Draft 6 recommends disclosure and a curve, because the cliff measured the wrong thing. Each step removed something the protocol would have had to decide. That is a direction, not a drift, and the reasons for the last step are these.

A threshold on size cannot distinguish a deception from a labelled choice. Sceptre's failure was concentration nobody could see; a per-provider token with one operator on the tin is concentration everybody can see, and a cliff would penalise the second for growing into exactly what it promised. A cliff forecloses a choice. A curve prices it.

A cliff needs an allocator. Something has to compute the target the cliff is measured against, and that something becomes the largest delegator on the network by proxy, the object of permanent parameter politics, and the largest selector of data providers if it ever binds enough stake. Those were Failure Modes 11.1, 11.2 and the second half of 6.9, and they were the strongest objections the document had to itself. A curve has none of them, because nothing decides.

Both designs are gamed by splitting identities, so neither can bind before Scope 0. The argument that had earlier drafts reject a curve as Sybil-vulnerable applied equally to the router and was never a reason to prefer it.

Under any reward-based instrument the cost lands on delegators, because the P-chain does not slash and the registry is the only other place to reach. Sceptre used that fact this week as the reason not to comply with a chill. The honest reading is that delegators are the ones who can move and the rate is their signal; a curve makes the signal continuous where a cliff made it binary and made the operator's argument for it.

### 4.2 Decision matrix

| Criterion | Scope 0 (disclosure) | Curve | Oracle (information) | Scope B (token) |
|---|---|---|---|---|
| Would have surfaced the Sceptre structure | Yes, in July | No | No | No |
| Reaches the pool, not only the registry | Indirectly: makes it visible | Yes, as a price | No | For migrated capital |
| Distinguishes disclosed from undisclosed concentration | It is the distinction | Prices both; disclosed can be chosen | — | — |
| Requires an allocator | No | No | It is one, non-binding | Yes |
| Consensus change | No | No; reward script | No | Yes (§5.2 ii) or FCC custody (§5.2 i) |
| Gamed by identity splitting | It is the fix | Until Scope 0 | Until Scope 0 | Until Scope 0 |
| Governance surface | Data format | Two parameters under FIP.11 | Formula parameters, non-binding | Custody envelope |
| Time to Songbird | One quarter | One to two quarters after Scope 0 | One quarter | Tied to a delegation FIP or a key set |
| Regulatory exposure (§11.10) | None | None | None | Higher |

Decision conditions, as proposals: ship Scope 0 on Songbird within one quarter of a decision and on Flare within two; put the percentage cap to FIP.02/FIP.05 in the same window; publish the oracle non-binding as soon as Scope 0 is live; switch the curve on only after two Songbird reward-epoch cycles with the operator mapping in place and Appendix D's split test passed. If by the end of Q1 2027 no contract-originated delegation FIP is in draft, treat B as closed until one is.

---

## 5. The Settlement Problem

This is the section a reviewer from the core team will read first, and it is the one where the sparring draft was silent.

### 5.1 Two chains, two delegations

Flare inherits Avalanche's architecture: a P-chain that holds validator stake and delegation, and a C-chain that runs the EVM. Staking rewards for P-chain stake are computed off-chain by a public script and distributed by C-chain contracts through a mirror (`PChainStakeMirror`). FTSO delegation is a different mechanism entirely: WFLR on the C-chain, delegated by percentage to a data provider, no lock, rewards every 3.5 days. FIP.16 unified the weight both carry in the FSP protocols and set P-chain stake at 5×.

A liquid staking pool on Flare has to do both. Sceptre's contract holds deposits on the C-chain, delegates WFLR to FTSO providers, and moves a portion to the P-chain to stake against validators. The second step is the problem. A C-chain contract cannot construct or sign a P-chain transaction. Somebody has to export FLR from C to P, issue `addDelegator` or `addValidator`, and later import it back. That somebody holds keys. In Sceptre's case, four accounts with DEPOSIT and WITHDRAW roles, granted on 15 July 2026, that moved roughly 140 million FLR out of the pool into self-bonds with no timelock, no cap and no notice.

The four keys are not a Sceptre design flaw. They are the only way a C-chain pool can reach the P-chain today. Every liquid staking token on Flare has a version of them.

### 5.2 Three ways to close the gap

**(i) Protocol-managed keys in Flare Confidential Compute.** FIP.16 Stage 2 already plans to run the block builder inside FCC. The same enclave model can hold the P-chain keys of the eFLR pool: keys generated inside a TEE, never exported, executing a fixed policy (export, delegate per router output, import at maturity) against attested inputs. This is the author's earlier "Ulysses at the mast" proposal applied to staking rather than treasury. It is buildable on the FCC roadmap and it does not change consensus. Its weakness is the enclave image: whoever can replace it controls the pool, so the attestation and upgrade path for the image is the actual custody question.

**(ii) A P-chain primitive for contract-originated delegation.** The P-chain recognises a delegation whose owner is a C-chain contract address, with the mirror running in reverse: the contract emits intent, the P-chain executes it, the mirror proves execution. This is the clean answer, and it is a consensus change that nobody has scheduled. Draft 1 read FIP.16's Stage 3 sentence, that "the technical implementation of how exactly staking is handled, where the funds reside, and concrete staking processes will change", as a window for this. That reading was too generous. Stage 3 changes who builds and proposes blocks; it says stake keeps counting as vote power and validators stay the security layer; it reopens staking mechanics without saying anything about a contract as delegator. The primitive is a separate FIP with its own scope, and it should be written that way so that it is not hostage to the builder roadmap's timing, and so that the builder roadmap is not asked to carry a change it did not propose.

**Mirror limit.** One further constraint neither route escapes. The C-chain mirror recognises stake to at most three validators per P-chain address for rewards and vote power. A protocol pool delegating across a hundred entities therefore cannot sit on one P-chain address; it needs a vault architecture of many addresses, each holding stake to at most three nodes, with reward accounting that aggregates across them. That is engineering, not research, but it sizes the work, it multiplies the key material under (i), and it is why "one enclave, one key" in Appendix A.2 is a simplification the core team would have to correct.

**(iii) Restrict eFLR to C-chain delegation.** A pool that only delegates WFLR to FTSO providers needs no P-chain keys at all. It also earns at 1× weight against 5× for P-chain stake, and it does nothing for validator concentration, because it never touches validators. This is not a liquid staking token; it is a delegation wrapper, and it is mentioned so that it can be ruled out.

The RFP asks the core team one question above all others: will (ii) be scoped as its own FIP, and what would it cost? Section 6.1–6.9 and Section 8 assume that either (i) or (ii) exists; Scope 0, the curve and the oracle assume neither.

Between (i) and (ii) the RFP takes a position rather than leaving it as a menu. The enclave route has a single point of failure in the image and its attestation chain, and a compromise there is a loss of the pool with no recovery path except a hard fork (§11.13). The primitive route has consensus risk, which is bounded by Songbird testing and by the fact that the network has shipped consensus changes through that path before, most recently the FIP.16 fork. A consensus change tested on Songbird is a risk the network has taken before; an enclave holding a multi-billion FLR pool is not. The primitive is the target; the enclave is acceptable only as a bridge with a published sunset. Appendix A sets out the minimum each option would have to specify for the core team to answer with more than "possible".

### 5.3 Liquidity: the exit queue

P-chain delegation locks for a chosen period, minimum two weeks, and cannot be withdrawn early. A liquid token over locked capital needs either an instant-redemption reserve (Sceptre's model, and the reserve the four keys drew down) or an exit queue with a redemption delay, or both. The sparring draft assumed instant, zero-fee redemption during a governance timelock; that is not achievable against locked P-chain stake without a reserve large enough to honour it.

The proposal: a laddered delegation schedule (delegations staggered so that a fixed share matures every epoch), an exit queue served in order from maturing delegations, and an instant-redemption reserve whose minimum size is a contract invariant, not an administrator's discretion. The ragequit right in Section 8 is the right to enter the queue at the head, not the right to instant exit, and its value depends on the ladder: priority in a queue whose next maturity is 180 days out is cosmetic, which is why the ladder length is a parameter with a ceiling, not a free choice.

Three things the sketch has to say to be a design. First, a discount policy. Under stress eFLR trades below its exchange rate on secondary markets, and the question is who may close that discount. Nobody with administrator access to the reserve; the invariant exists to make that impossible. External market makers, yes, that is what a discount is for. FIRE, no: a protocol buying its own pool's token to defend a peg is the protocol becoming a market maker in its own liability, and it is excluded in writing. Second, a griefing model. An attacker can inflate the queue, drain the reserve to its floor, push the secondary discount, and then present governance with a demand for a "reserve patch". The defence is that the invariant is in the immutable core and the emergency powers in Section 8 cannot reach it; the cost of the attack is the attacker's own exit at a discount; and the queue serves in order, so the attacker exits last of its own tranche. Third, numbers. Appendix C runs 5%, 10% and 20% outflows over one and thirty days against 90-, 180- and 365-day ladders with a 5% reserve. The short version: a 5% reserve absorbs a 5% one-day run outright, a 10% run clears in five to twenty days depending on the ladder, a 20% run in fifteen to fifty-eight. A ladder longer than 180 days turns every stress event into a two-month queue, and the reserve size and ladder length should be set from that table, not from the number 5.

Sceptre's buffer was advertised at about 50 million FLR and was the thing the four keys drew from. eFLR's reserve has one job beyond liquidity: to demonstrate that "invariant" does not mean "there is another function the administrator can call".

---

## 6. Architecture

### 6.1 Deposit and token

Users deposit FLR (or WFLR) and receive eFLR. Two accounting models exist. A rebasing token (balance grows) is simpler to reason about and breaks most DeFi integrations. A value-accruing token (balance fixed, exchange rate to FLR rises, sFLR's model) is what every lending market, AMM and yield-stripping protocol on Flare already integrates. eFLR should be value-accruing. This is not a small choice: it determines whether Spectra, Kinetic and Enosys can list eFLR on day one by cloning their sFLR configuration.

### 6.2 Capital sources

Two inflows. User deposits, which are the point. And, as an option the RFP puts on the table rather than assumes, protocol-owned capital: FIRE's mandate already ranks "rewards to FLR validators and stakers" as its second priority, and FIRE's captured FLR could be deposited into eFLR rather than distributed. The effect would be to seed the pool and to give FIRE a yield-bearing FLR position. Two cautions. FIRE's assets are mixed (stablecoins, FXRP, FLR), so only the FLR leg qualifies. And a pool part-owned by the Foundation's revenue entity is a pool whose neutrality will be questioned; Section 8 addresses whether the governance envelope can carry that.

The "deflationary flywheel" claim from the sparring draft is dropped. Locking FLR in a pool changes float, not supply. The fee in Section 7.2 burns a fraction of yield that is small against 3% inflation. eFLR is a supply sink in the FIP.16 sense (excluded from the inflatable base while locked) and nothing more, and the document should not say more.

### 6.3 The allocation oracle

The oracle is the formula a neutral delegator would follow. Every reward epoch (3.5 days) it computes and publishes a target allocation across eligible entities from public inputs. Under Scope B it is what the protocol pool executes, subject to lock schedules. Under everything else in this document it is information: a benchmark depositors can hold their pool against, and a reference the curve's parameters are set beside. The rest of Section 6 specifies it in full because Scope B needs it in full, and because a benchmark nobody can recompute is not one.

Target allocation per entity *i*:

```
A_i = min( W · (T_i · Y_i) / Σ_j (T_j · Y_j) ,  C_max,i ,  K_i )
```

where W is the pool's delegable stake, T is a trust weight, Y is a net-yield weight, C_max,i is the per-entity cap from 6.7, and K_i is the entity's remaining delegation capacity under FIP.05's 15× factor and the 300 million node maximum. Surplus above any entity's cap or capacity redistributes to the next by the same weights. Without K the router issues instructions the P-chain rejects: free delegation space exists in aggregate (about 8.6 billion on 5 September 2026) and individual entities are full.

The formula is a target, and a target is not an instruction. P-chain delegation locks for two weeks to a year, so the pool cannot reach its target every 3.5 days. Three constraints turn the formula into a schedule. A rebalance budget: in any epoch, the router may move only capital that matured that epoch plus reserve above the invariant, and nothing else. A maximum step: no entity's allocation moves by more than a fixed share of the pool per epoch, so that a change in T or Y produces a drift, not a jump. A tracking-error bound: the distance between actual and target allocation is published every epoch, and if it exceeds a threshold for more than a set number of epochs, that is a monitoring event under Section 11, not a reason to break the lock schedule. The on-chain state is the delegation ladder; the formula is what the ladder is steered toward. Delegation is per entity, summed over that entity's registered nodes, because the entity is what the registry knows and the node is what the P-chain knows; the router must reconcile both.

Three terms, used consistently from here:

| Term | Definition | Known to | Role in the oracle and the curve |
|---|---|---|---|
| Node | One validator with its own NodeID and stake | P-chain | Unit of delegation execution |
| Entity | Registered identity, one to four nodes | C-chain registry | Unit of scoring (T, Y) and of the cap C_max |
| Operator | The organisation behind one or more entities | Scope 0 mapping, once it exists | The unit the curve prices (6.10); target of the correlation rule (6.8) |

The router scores and caps entities, executes against nodes, and applies the correlation rule at entity level: an entity that spreads four nodes across four hosts does not thereby escape a penalty it shares with another entity on any one of them.

### 6.4 Eligibility (hard filters)

An entity is eligible if, over the evaluation window, it has met the FIP.05 uptime requirement (80%; the RFP proposes the oracle use a stricter 95%), has been rewarded by the FSP for FTSO data provision in every epoch of the window (which is what "good enough prices" means in FIP.05 terms and is already computed on-chain), holds no active chill under FIP.02, and has a self-bond meeting the P-chain minimum. There is no slashing on Flare's P-chain; "slashing history" in the sparring spec is replaced by chill history, which is the network's actual sanction record.

### 6.5 Trust weight T_i

New entities enter at T = 0.1 and rise linearly to T = 1.0 over 50 reward epochs (about six months) of continuous eligibility, and for those 50 epochs their cap is one fifth of C_max. A higher starting weight was considered and rejected: at the pool sizes this document contemplates, half weight after one eligible epoch is a large delegation to a party the network has watched for four days. The purpose is to make Sybil entry expensive in time, since a 1 million FLR self-bond does not make it expensive in capital. A window of 30 epochs (about 105 days) governs penalties: each epoch of ineligibility inside the window deducts 0.2 from T, floored at 0; eligibility restores T by 0.05 per clean epoch. An entity that goes dark for one epoch loses a fifth of its weight for four epochs and is back at full weight within a month. An entity that is chilled loses eligibility for the chill's duration and re-enters at T = 0.5. No lifetime exclusions; the router is not a court.

The obvious attack is identity splitting: an operator at C_max registers a second entity, waits out the ramp, and after six months collects delegation under two names. Two things bound it, and neither is the ramp. FIP.05's delegation factor caps total stake on a node at 15× its self-bond, so a fresh entity with the 1 million FLR minimum can receive at most 14 million in delegation from anyone, router included; scaling the attack means scaling self-bond, which is capital the operator has to lock. And the cap in 6.7 is computed on the entity's share of network stake, so a second entity that shares infrastructure with the first shares its cap under 6.8. A reviewer proposed tying the ramp to self-bond size directly (T rising more slowly below a 5 million FLR bond). The RFP does not adopt it: the delegation factor already prices Sybil entry in capital, and a bond-weighted ramp would slow exactly the small honest operators the baseline exists for. What the ramp buys is time, and time is the one input a well-funded attacker cannot buy.

### 6.6 Net-yield weight Y_i

Flare's validators are data providers, and their reward rate varies with data quality. A router that ignores that pays bad data providers to exist. Y_i is the entity's realised net reward rate to delegators over the trailing 10 epochs, divided by the network median:

```
Y_i = (gross reward rate_i × (1 − fee_i)) / median_j(gross reward rate_j × (1 − fee_j))
```

Fee is the entity's declared fee, floored at 20% by FIP.16. An entity charging 35% competes against one charging 20% on net yield and loses delegation to it; an entity whose FTSO submissions earn more competes on gross and wins. Y is clamped to [0.5, 1.5] and computed on a trailing ten-epoch mean, so that one exceptional epoch moves an entity's weight by at most a tenth of its excess and the router remains a baseline, not a momentum trader. Yield-hopping, where delegation chases last epoch's winner, is a failure mode of every reactive allocator, and the window and the clamp are the two instruments against it; if either proves too loose on Songbird, both are parameters.

Y has a second problem the clamp does not solve. Once the pool is large, its own allocation is a determinant of the yields it measures: delegation raises an entity's signing weight, which raises its FSP rewards, which raises its Y, which raises its delegation. The clamp bounds the speed of that loop, not its existence. Two mitigations, both partial: Y is computed on reward rate per unit of stake, not on reward volume, which removes the first-order effect; and the cap in 6.7 bounds how far any one entity can ride the loop. The residual is a real limit of a reactive allocator at scale, and it is one reason the router should publish its allocation for several epochs before it binds anything.

Under Scope B this is what answers the objection that a protocol pool socialises the validator set: delegation from the protocol pool follows three numbers every entity can see and move, and an entity that improves its feeds gains delegation next epoch from a delegator that cannot be lobbied. As a published oracle it does something smaller and still useful: it lets a depositor in any pool see how far that pool's choices sit from a neutral benchmark, which is a fact no pool publishes today.

### 6.7 Cap C_max

The cap is the point of the whole design and the parameter most likely to be fought over. The sparring draft proposed 3% of the pool per node. That is the wrong denominator. A pool that is 40% of the network with a 3% per-node cap still hands 1.2% of the network to a node in one instruction; and a cap on the pool's share says nothing about the entity's total share once its other delegators are counted.

The proposal: C_max is expressed as a share of total active network stake, per entity, inclusive of the entity's stake from all sources as the reward script already sees it, and set so that the router never pushes an entity across the binding percentage cap this RFP assumes governance will adopt in FIP.02/FIP.05. If that cap is 5%, the router's C_max is 5% minus the entity's non-router stake. The router then does with code what the Management Group cannot do with a forum: it makes the largest delegator on the network incapable of concentrating.

### 6.8 Correlation

Eight of Sceptre's nodes sat in one /24 on one host. A cap per entity does not see that two entities share a rack. The right rule is that entities whose nodes share an autonomous system, hosting provider or /24 with another eligible entity share one cap between them for the shared portion. The right rule is not yet a consensus input. Host and AS attribution is off-chain data (Catenalytica publishes it), it is contestable, and a router that reads it is a router whose allocation can be moved by whoever edits a hosting record. Until an oracle-grade source exists, the correlation rule is advisory: computed and published alongside the allocation, reported quarterly under Section 11, and not part of C_max, which is the wrong place for data the chain cannot verify.

### 6.9 Two yields, two routers

A Flare liquid staking pool runs two machines, and this document has so far specified one. P-chain stake earns validator rewards and carries 5× signing weight; it is locked. WFLR on the C-chain earns FTSO and FSP rewards at 1× and is liquid. A pool decides how much of its capital sits in each, and Sceptre decides it today by discretion. The RFP proposes: the split is a parameter, initially set so that the liquid leg covers the reserve invariant plus the next epoch's expected redemptions and everything else is exported to the P-chain, because 5× against 1× is not a close call for yield or for security. The C-chain leg is delegated by the same entity weights as the P-chain leg, so that the router does not run two contradictory selections; a separate FTSO-only formula is possible and not proposed. Rewards on both legs are claimed by the contract each epoch and compounded into the exchange rate, with no claim role held by any party.

The consequence applies to Scope B only, and it is one reason B is last: if eFLR reached 30–50% of stake, the protocol pool would be not only the largest delegator but the largest selector of FTSO data providers, and the oracle's entity weights would decide a large share of which prices the network publishes. Under Scope 0 and the curve nothing selects, which is the point of preferring them.

### 6.10 The curve

Let s_i be operator i's share of active network stake, summed over every entity the Scope 0 mapping attributes to it and every node under those entities, as the reward script already sums per node. Let r be the reward rate a unit of stake would earn with no adjustment. The rate paid on stake attributed to operator i is:

```
r_i = r · f(s_i),   with  f(s) = 1              for s ≤ s0
                           f(s) = (s0 / s)^k     for s > s0
```

Two parameters. s0 is the share below which nothing happens; the proposal is 3%, so that today's largest entity (Bifrost, 1.14 billion, about 5.3%) and the largest pool's own-node share sit above it and the median entity sits well below. k sets the slope. At k = 1 an operator's total rewards stop growing at s0: every unit of stake above the line earns exactly enough less that the operator's aggregate is flat, which is what the existing 5% vote-power cap does per validator with a kink instead of a curve. At k = 0.5 total rewards keep growing above s0 but at a diminishing rate. The proposal is k = 1 with s0 = 3%, which makes the curve the existing cap's logic applied per operator and started earlier; the network already lives with the k = 1 rule and nobody has called it confiscation.

Three properties, stated so they can be argued with. It is continuous: an operator crossing s0 by one unit loses almost nothing, and there is no epoch in which a slice of stake earns zero. It is neutral about selection: it does not care which entities a pool chooses, only how much of the network any one operator carries. And it is chosen: a depositor who wants a single operator at 8% can have it and sees the rate; a pool that advertises spread and delivers 98% behind one operator is paying the curve on capital it told depositors was diffused, and the Scope 0 mapping is what lets a depositor see the difference.

What it does not do. It does not touch a pool's C-chain WFLR delegation to FTSO providers unless the reward script applies it there too; the proposal is that it does, on the same operator share, because the FIP.16 signing weight already unifies the two legs. It does not bind before Scope 0 exists, because an operator who can split identities can reset s_i; that is the whole argument for sequencing. And it does not reach an off-chain kickback, which no design here does.

Who pays. The delegators on the stake above s0, at a rate they can see in advance. The operator pays only through the fee it takes on rewards that are now smaller. This is the same incidence as a chill and as the retired cliff, and it is the incidence of every reward-based instrument on a chain without slashing. The difference is that a curve is a published price and a chill is a verdict.

---

## 7. Economics

### 7.1 The base rate

The pool's gross yield is the stake-weighted average of what its delegated entities earn, less their fees (20% floor), less the protocol fee below. At today's parameters an sFLR holder sees roughly 7–8% gross on the pool, of which Sceptre takes 10% and the network's 3% inflation absorbs most of the rest in real terms. The author's estimate of Sceptre's numbers: about 228 million FLR a year of pool rewards on 2.384 billion, about 23 million of which is service fee.

eFLR's advantage over that is not the fee, and the arithmetic should be on the page. Call the gross reward rate on delegated stake g. Under Scope B the protocol pool pays the 20% floor on everything, leaving 0.80g, and a 5% protocol fee on that leaves 0.76g to the depositor. A pool that self-validates a third of its capital (Sceptre's proportion) and passes the self-bond rewards through earns g on that third and 0.80g on the rest, 0.87g before its own 10% fee, 0.78g after. Under the same gross, eFLR pays the depositor about two and a half percent less than a self-validating private pool, and more if the private pool keeps the self-bond rewards for its operator instead. That gap is the price of the protocol pool paying every entity the floor and being incapable of self-bonding. eFLR does not win on yield; it wins on not being the counterparty that can take the reserve, and Section 10.7 says what follows for migration.

### 7.2 Protocol fee

A fee on pool rewards, proposed at 5% for discussion; Sceptre charges 10%, typical LST fees run 10–20%, and the right number is whatever covers the reserve and leaves eFLR's net rate above the private pools' without a self-bond subsidy. Two destinations, and the sparring draft's 50/50 split is replaced with something the numbers can carry.

The first destination is the instant-redemption reserve, until it reaches its invariant size (proposed: 5% of pool). Once the reserve is full, the fee goes to the second destination, and here the RFP has to name a conflict it created. FIRE is administered by the Foundation. A fee from eFLR to FIRE is, in practice, a fee to the Foundation, and it gives the Foundation a direct financial stake in eFLR's growth. Every parameter in Section 6 that makes eFLR more attractive to depositors (a looser uptime filter, a softer Y clamp, a higher cap) raises that fee. If the same Foundation also holds the custody keys, the party that sets the router's neutrality is paid for the router's size. That is not a hypothetical; it is the structure.

The fee's destination is therefore conditional. If the custody condition in Section 8 is met, and no single party including the Foundation can replace the enclave image or the upgrade key, the fee may go to FIRE under its existing mandate, because FIRE's priorities two through four are the ones an eFLR fee should fund. If the custody condition is not met, the fee is burned. And in either case one combination is excluded: FIRE may not seed the pool (Section 6.2) while FIRE receives the fee. A protocol that is a depositor in its own pool and the recipient of its fees has closed a loop in which its neutrality is not a matter of keys at all. One or the other; the RFP's preference is the fee, because seeding is the one that cannot be undone. The 5% itself stays a discussion number until the reserve simulation in Appendix C has been run on Songbird data; it is not cemented here. Burn benefits every FLR holder equally and gives no party an interest in the pool's size. The RFP does not propose a separate "insurance fund". Flare's P-chain does not slash, so there is no principal loss to insure against from validator conduct. The arithmetic for the risks that remain is short: a 2 billion FLR pool at 7% gross yields about 140 million FLR a year, of which 5% is 7 million; an enclave compromise loses the pool. A fund that claims to cover a loss three hundred times its annual inflow is marketing, and "principal-protected" does not appear in this document. What a compromise does get is §11.11.

### 7.3 Who earns what

The depositor earns the base rate net of 20% entity fees and 5% protocol fee, holds a token the counterparty cannot drain, and gives up the possibility of the above-market yield a self-validating pool can offer. The entity earns the 20% floor on router delegation, allocated by uptime, data quality and fee discipline rather than by relationship with a pool operator; small entities that perform get delegation they cannot market for today. The protocol earns a fee stream into FIRE and, more importantly, a validator set whose largest delegator is incapable of concentration by construction. The private pool operator loses the yield edge from self-bonding and the discretionary selection business, and keeps distribution, integrations and strategy layers.

### 7.4 What the curve does and does not do to entities

Nothing to the 20% floor. The curve adjusts the rate on stake by operator share; the floor is the entity's cut of whatever that rate produces. An entity at 1% of the network is untouched. An operator at 8% keeps its floor on rewards that are smaller per unit above 3%, and its delegators see why.

Nothing to marketing. Earlier drafts, defending the router, argued that it made marketing to stakers worthless above a threshold and that this was aligned with FIP.16's shift from inflation to organic yield. The reviewer who pushed back on that asked whether it was Flare's stated intent or the author's inference, and the split is in Appendix F: FIP.16 says where infrastructure income will come from and says nothing about what a validator should be or how it gets funded. With the router retired the argument is moot and the document withdraws it. Under a curve, an entity that raises its bond by being trusted, that markets, teaches and ships, earns delegation exactly as it does today, and pays the curve only if it becomes a large share of the network by doing so. The operator starting tonight with a machine and no money is below s0 by definition and nothing here reaches him. The moat objection that the router had to answer does not arise, because nothing allocates.

What the curve does to the largest operators is the point. Bifrost at about 5.3% and Sceptre's own-node share pay a lower rate on the part above 3%, and their delegators can see it, compare it with the oracle's benchmark, and move or stay. That is the entire mechanism. It has no opinion about which entities are good.

---

## 8. Governance Envelope

The sparring conversation's own objection: enshrining the pool moves the bottleneck from four private keys to whoever can change the contract. On Flare that is, today, the Foundation, through the governance timelock on every system contract. The FIRE precedent is instructive and not reassuring: FIRE is administered by the Foundation with a "negative governance" veto that requires 50% of the inflatable supply to vote, a threshold no vote has reached.

The envelope this RFP proposes has four parts, and the fourth is the one that matters.

**Immutable core.** Deposit, redemption, exit queue, the reserve invariant, and the cap formula are non-upgradable. If they are wrong, the pool is wound down and redeployed; it is not patched.

**Parameter set under timelock.** For Scope B: T ramp length, penalty size, Y clamp, uptime threshold, fee, reserve size, correlation rule: adjustable, with a 21-day on-chain timelock and public diff before effect. For the curve, two parameters, s0 and k, under FIP.11's methodology and the same timelock; a curve with a dozen parameters is a router by another name.

**Ragequit during timelock.** Any pending parameter change opens a window in which eFLR holders may enter the exit queue at the head, ahead of ordinary redemptions, served from the reserve and maturing delegations. It is not instant exit; Section 5.3 explains why it cannot be. It is guaranteed priority.

**Custody outside the Foundation.** This is the part the sparring draft's dual-veto does not solve. A veto over parameters is worth little if the enclave image, the router's data feeds and the upgrade key are held by the same party. The four buckles from the author's FIRE essay apply verbatim: the upgrade key, the mandate governance, the enclave image, the input feeds. Each must have a documented holder, and for at least the enclave image and the upgrade key the holder must not be the Foundation alone. A 2-of-3 across Foundation, an elected entity committee (FIP.16's own joint-governance mechanism, Songbird-then-Flare election) and a time-delayed community key is the minimum. If the Foundation will not accept that, there is no Scope B, and the document is Scope 0, the cap and the curve; the four keys stay with the pools, bounded by the contract terms the earlier dispatch asked for and visible under Scope 0.

**Transition.** A 2-of-3 that does not exist at launch is a promise, and Section 3 of the author's FIRE essay already said what promises are worth. The proposal is staged so that each stage is a precondition for the next, not a hope after it. At launch, on Songbird, the Foundation holds the keys alone, under the 21-day timelock and the ragequit right; this is acceptable because Songbird is the test and the amounts are SGB. Before any Flare deployment, the entity committee exists: elected by the mechanism FIP.16 §4.5.1 already specifies (a Songbird vote to shortlist eight, a Flare vote to seat four) but without FIP.16's precondition of a 50%-of-supply vote to unlock it, which has never been met and was designed not to be. Before the pool exceeds a threshold share of active stake (proposed: 10%), the community key exists: a time-delayed multisig whose signers are elected by eFLR holders, with the power to block a parameter change or an image replacement and no power to initiate either. If the second or third stage is not in place when its trigger arrives, deposits close until it is. Closing deposits is the enforcement; there is no other.

**Custody matrix.** Section 8 is an essay until it is a table. The one below is the proposal; every cell is a decision the Foundation can accept, reject or replace, and a cell left blank at launch is a reason not to launch.

| Asset or action | Who signs | Delay | If the key is lost |
|---|---|---|---|
| Router parameters (T, Y, filters, cap, ladder) | 2-of-3: Foundation, entity committee, community key | 21 days, epoch-boundary effect | Status quo holds; missing party is re-elected |
| Enclave image hash (route i) | 2-of-3, with the community key required | 21 days + one full ladder before old key retires | Old image continues; no new deposits until replaced |
| Reserve invariant, exit queue, cap formula | Nobody; immutable | — | Wind-down path below |
| Pause new deposits and new instructions | Any one of the three | Immediate; expires after two epochs unless renewed by 2-of-3 | — |
| Move capital | Nobody; only the bridge, only per allocation | — | — |
| Wind-down | 2-of-3 | 21 days; then deposits close, ladder runs off, redemptions at exchange rate | — |

Who the parties are, concretely. The Foundation is the Foundation. The entity committee is the four seats FIP.16 §4.5.1 already describes, elected by SGB shortlist and FLR vote, seated without the 50%-of-supply precondition; its members are entities the router pays, which is a conflict, and the reason it holds a veto and not an initiative. The community key is a 7-of-12 multisig whose signers are elected annually by eFLR holders weighted by balance, generated in a public ceremony, rotated at each election, with a 14-day delay on every signature. The FCC image, if route (i) is used, is a reproducible build whose hash is published on-chain by the 2-of-3 before attestation is accepted, with a written policy that a CVE in the TEE triggers a pause, not a patch, until a new image passes the same path. Immutable core and "wind down if wrong" is an event at 30–50% of stake, not an upgrade path, which is why the pause power exists and why wind-down is specified as a ladder run-off with redemptions at the exchange rate, not a redeploy.

**Deadlock.** Three parties will disagree. The rule is that the status quo wins: any change to parameters or to the enclave image needs two of three, so one party can block, and a block is not a crisis, it is the design working. Where all three want a change and differ on the value, the most conservative proposal takes effect (the lowest cap, the strictest filter, the longest timelock), and the others may propose again after one reward epoch. Where the disagreement is about an emergency (a live exploit, an image with a known bug), the only emergency action available to any single party is to pause new deposits and new delegation instructions; existing delegations mature on schedule and the exit queue keeps running. Nobody can move capital alone, in an emergency or otherwise. That is the whole point of the envelope, and an emergency power that could override it would be the four keys again.

---

## 9. Composability

eFLR is a base asset. Everything below it in the stack (fixed-term lending, options collateral, yield stripping) exists or is proposed elsewhere, and this section decides what the protocol builds and what it leaves.

The sparring draft proposed native PT/YT splitting in the protocol. This RFP recommends against it. Spectra already runs yield tokenisation on Flare against sFLR, with live pools across maturities, and the Fixed-Term Lending RFP already specifies PT-as-collateral on Spectra's rails. An enshrined splitter duplicates a live product, adds contract surface to the core, and forces the protocol to maintain a maturity calendar. What the protocol should do instead is make eFLR trivially strippable: value-accruing accounting, a clean exchange-rate oracle, no transfer hooks, no rebasing. Spectra then lists eFLR as it listed sFLR, and PT-eFLR becomes the collateral the lending RFP wants: a claim on protocol-custodied FLR with no operator key risk, which is the property that makes it institutional-grade rather than merely fixed-rate.

The author's conflict is on the table here: the recommendation favours a protocol the author uses and a market the author wants to exist. The counter-argument, that enshrined stripping removes a dependency on a third party, is real, and Section 10 keeps it open.

**FAssets.** The largest consumer of FLR collateral on the network is not a lending market; it is the FAssets system, where FLR sits in agents' collateral pools behind FXRP. eFLR in that role would replace an asset with operator key risk (sFLR) with one without it, which is an argument FAssets governance should hear, and it comes with a liquidity condition: an FAssets liquidation that receives eFLR has to be able to turn it into FLR faster than the exit queue allows. Two ways to meet that. A haircut on eFLR's collateral value sized to the secondary-market discount observed on Songbird under stress, which is the honest way and needs no new mechanism. Or a liquidation lane in the reserve, ahead of ordinary redemptions and behind ragequit, capped at a share of the reserve the invariant can spare. The RFP proposes the first and leaves the second to the FAssets team, and it does not propose collateral factors; those are FAssets governance's to set against data that does not yet exist.

---

## 10. Open Questions

Ordered by how much the answer changes the design.

1. **Will the core team scope contract-originated P-chain delegation as its own FIP?** It is not on the Stage 3 path (§2.4, §5.2) and should not be made to wait for it. Until it exists, Scope B depends on FCC-managed keys and the enclave image becomes the custody question.
2. **Can the oracle's inputs be made oracle-grade?** Uptime and FTSO reward data are on-chain; host and AS attribution are not. The correlation rule (6.8) needs a source the protocol can verify or it stays advisory.
3. **Does the registry refuse a fifth node?** The earlier dispatch could not confirm it. If the registry enforces the count, Scope 0 can read entity membership from it; if not, the mirror service is the source and phase 3 is a prerequisite.
4. **Where does the curve start and how steep is it?** s0 = 3% and k = 1 are proposals that make the curve the existing 5% cap's logic per operator, started earlier. A lower s0 binds more operators; a lower k prices concentration more gently. The RFP asks for the parameters to be set from the Scope 0 data, not before it.
5. **Should FIRE seed the pool?** Section 6.2 puts the option forward and names the neutrality cost. The answer depends on Section 8's custody outcome.
6. **Native stripping or Spectra rails?** Section 9 recommends rails. The dependency argument against it deserves a written answer from the Foundation, not from this author.
7. **What happens to sFLR?** Three paths, and the RFP should be plain about which ones work. A soft path (eFLR launches, sFLR stays, a conversion at the snapshot exchange rate with a small bonus for early movers) is politically costless and, on the Cosmos and Ethereum evidence, does not move capital; holders stay where their integrations are. A priced path (the curve: sFLR's own-node share above s0 earns less, and its delegators see it) removes nothing by force and leaves Sceptre its 10% service fee, its front-end and its integrations; what it changes is that the self-bond edge is now paid for, in the open, and a rational Sceptre may prefer that to a cap it has to route around. A third path, freezing or force-converting the sFLR contract, is excluded here and should be excluded in writing by the Foundation: it would establish that a protocol can seize a private contract's deposits by vote, and that precedent is worth more to an attacker than any pool. The honest statement is that under Scope B alone eFLR does not reach critical mass, and that under Scope 0 and the curve it does not need to; the question of what happens to sFLR becomes what sFLR's depositors do once they can see the operator share and the rate on it. The arithmetic of the self-bond edge is in 7.1 (about 0.87g to 0.80g before fees); the curve prices that edge instead of removing it.
8. **Songbird first?** Every consensus-touching FIP has shipped on Songbird first. Scope 0 and the oracle can run on Songbird within a quarter of a decision; the curve after two Songbird cycles with the mapping live; the token (Scope B) not before it has run through at least two Songbird reward-epoch cycles with the exit queue under load.
9. **What is an operator?** The curve is keyed to operator share, and an operator is not an on-chain object. The proposal: an operator is any set of entities whose self-bonds were funded from, or whose reward owners are, addresses that share privileged roles on a common contract or a common controlling key, and the burden of showing separateness lies with the operator. That is what Scope 0 has to publish and what the reward script has to read; without it the curve is arbitrage.
10. **What does it cost?** Engineering effort for the P-chain primitive, FCC provisioning for the enclave route, audit scope for the immutable core, and the reserve's opportunity cost. This RFP has no basis for those numbers and does not invent them. The decision matrix in 4.1 is incomplete without them, and the request is that the core team fill that column before the scope decision, not after.
11. **Will FAssets governance accept eFLR as pool collateral, and on what haircut?** Section 9 argues it should; the number depends on Songbird stress data that does not exist yet.

---

## 11. Failure Modes

Hardest first. Before the list, who measures: every test below carries a date and a threshold, and a Foundation that measures its own project will find it succeeding. The proposal is that the metrics (largest operators' shares of active stake, the paid rate per operator against f(s_i), oracle-versus-actual distance per pool, and under Scope B exit-queue depth and secondary-market discount) are published as time series by a party that does not hold keys, on the pattern Flare Metrics and Catenalytica already follow, quarterly, on the forum. Three consecutive missed quarterly thresholds open a review in which the three custody parties of Section 8 must, within one reward-epoch cycle, choose one of: adjust parameters, change scope, or wind down. Under Scope B, eFLR's own share is a separate metric and a weaker one than whether the largest operators' shares moved.

**11.1 The enshrined pool is the largest pool (Scope B).** If eFLR works it becomes 30–50% of active stake, and the largest delegator on the network is a contract whose parameters are set by governance and whose keys are held by whoever holds them. Every objection to Sceptre's 11% applies at three times the size. The design's defence is that the router cannot concentrate (6.7) and the keys cannot be used discretionarily (8). Both defences are exactly as strong as their implementation and no stronger. If the four buckles are not closed, this RFP has proposed a bigger Sceptre with a Foundation logo. Test: before mainnet, an independent review of the enclave image and key holders, published; if any single party can replace the image, do not ship Scope B.

**11.2 The allocator becomes the politics (Scope B).** Under Scope B every parameter in Section 6 is a decision about who gets delegation. Entities will lobby for the uptime threshold, the Y clamp, the correlation rule. The forum fight about Sceptre's identities becomes a permanent fight about T and Y. Mitigation: parameters under timelock with ragequit, so that a lost fight is an exit; and a rule that parameter changes take effect only at reward-epoch boundaries with a 21-day notice. This does not remove the politics. It gives it a schedule.

**11.3 The proxy problem survives.** A curve per operator does not stop a cartel of twenty operators from holding 60% of the network between them at 3% each. Section 6.8's correlation rule catches shared infrastructure; it does not catch shared ownership on separate infrastructure. The document should not claim otherwise. The honest statement: disclosure makes the cartel visible if it shares infrastructure or funding, and a curve prices each member separately; collusion on separate infrastructure with separate money is left where every proof-of-stake network leaves it, at the limit of what code can see. An identity-staking rule, in which an entity bonds capital against its declared independence and loses it when a later-established breach shows otherwise, is the next instrument. Its buildability on Flare's P-chain is unresolved and it is not in this RFP; it is announced here as the follow-up so that C_max plus a correlation report is not mistaken for a solution to a problem it only halves.

**11.4 Nobody migrates (Scope B).** sFLR is integrated in every venue on Flare, has a liquid market, and can offer above-oracle yield by self-validating. eFLR launches with a lower yield and no integrations. Capital does not move for safety it has not been made to feel. Under Scope B alone this is likely; the Cosmos LSM's 25% cap was never binding because private ATOM liquid staking never reached it. Under Scope 0 and the curve migration is not the goal; visibility and price are, and the test is whether the largest operators' shares move once their delegators can see the rate. Test: eighteen months after launch, eFLR below 10% of liquid-staked FLR under Scope B is a failed launch; for the curve, the metric is the largest operator's share of active stake, and a share that has not moved in eighteen months means s0 or k was set too gently.

**11.5 The settlement gap is not closed.** If neither FCC-managed keys nor a P-chain primitive ships, Scope B is a private LST run by the Foundation, with the same four keys under a different name. In that case ship Scope 0, the cap and the curve, and say so.

**11.6 Liquidity crisis.** A run on eFLR during a market drawdown empties the reserve, the exit queue extends to the longest outstanding lock (up to a year), and eFLR trades at a discount on secondary markets. This is not a failure of the design; it is what a liquid token over locked stake does under stress, and every LST has done it. The failure is if the discount is used as an argument for administrator intervention in the reserve. The invariant in 5.3 exists to make that intervention impossible, and the document should expect the demand for it.

**11.7 The 20% floor collapses to the floor (Scope B).** Under Scope B the protocol pool delegates by net yield and makes 20% the ceiling. If the network's intention in FIP.16 was to allow entities to price above the floor, B contradicts it. The curve does not touch the floor (7.4).

**11.8 Ethereum was right.** The strongest external argument is that enshrinement was rejected on the network with the most research behind it, and that the rejection was on principle: a protocol that selects validators has taken a political role. Draft 6 largely concedes it: the recommended instruments select nothing. Disclosure publishes facts and a curve publishes a price; neither takes a political role, and the analogy to FIP.16 is now only that a rule in code replaced a rule in a forum. The objection survives against Scope B, and it is one more reason B is last.

**Threat model.** The failure modes below are the ones the design is built around. Appendix E lists the adversaries and what each can reach; the short list is TEE compromise, poisoned host data, governance capture of Y, queue griefing, and reward-script desynchronisation between C-chain and P-chain.

**11.9 The inputs are wrong.** The oracle allocates, and under Scope B the pool delegates, on FSP reward data and uptime; the curve reads operator shares from the Scope 0 mapping. If the FTSO layer is compromised, a Scope B pool delegates toward whoever is gaming it, at scale, every epoch; if the mapping is wrong, the curve prices the wrong operator. Two limits on that. A compromised FTSO is a compromised network; the router is not the largest thing that breaks, and a document that solved it here would be overclaiming. And the router can be made to fail closed: if any entity's inputs move more than a bound in one epoch, or the network median moves more than a bound, the router issues no new instructions that epoch and existing delegations mature on schedule. A pool that pauses is a delegator that stops, which is what Sceptre's four keys could not be made to do; a curve whose input is disputed falls back to the last accepted mapping until the dispute is resolved under Section 8. Cross-checking against a second data source is desirable and, for host attribution, not yet oracle-grade (Open Question 2); the fail-closed rule does not depend on it.

**11.10 Regulators read "protocol-issued yield-bearing token" and see a fund.** eFLR is a claim on a pool managed by a published algorithm with a fee. Under MiCA and under most securities analyses that description is close enough to a collective investment scheme to require an answer, and the answer "there is no issuer" is weaker for a Foundation-deployed contract than for a private one. Scope 0 and the curve carry none of this; they issue nothing. The RFP's position is that a written legal analysis for the EU, UK and US precedes any Scope B deployment on Flare, and that its absence is a reason to ship the instruments alone, not a reason to ship B and hope.

**11.11 The token is the Foundation's.** A protocol-owned LST with the Foundation in the custody chain is a different object from sFLR in one more way than the regulatory one: its brand is the network's. A depeg, a queue, a compromised image are not a startup's failure, they are Flare's, on the network's own name, with the network's own treasury adjacent. sFLR can fail and Flare survives it. eFLR cannot fail without the network wearing it. That is a reason to ship disclosure and a price, whose failure is a parameter set wrong, before B, whose failure is a token that did not hold.

**11.12 The curve is too flat, or too steep, or gamed.** Too flat and it prices nothing: operators at 8% pay a rate their delegators do not notice and concentration continues. Too steep and it becomes the cliff by another route, with the same incidence on delegators and the same argument against it. Gamed, and the game is identity splitting, which Scope 0 exists to close; if the mapping is wrong, the curve is applied to the wrong operator and the operator with the best lawyers pays least. Test: on Songbird, an operator that splits into three entities must see no change in its attributed share; if it does, the curve does not ship on Flare.

**11.13 The enclave is broken.** Under §5.2 (i) the keys live in an FCC image. If the image is compromised and the keys extracted, the P-chain delegations can be redirected as they mature and the reserve can be drained. There is no fund that covers this (§7.2), the loss falls on eFLR holders, and recovery is a hard fork or nothing. This is the argument for the primitive over the enclave, stated as a failure mode so that it is not lost: a route whose failure is total is a bridge, not a destination. Under (ii) the equivalent failure is a consensus bug in the primitive, which is why B waits for two Songbird cycles.

---

## 12. What Ships First

Disclosure the chain publishes, on Songbird within a quarter and on Flare within two: entity identity, operator mapping, self-bond source, reward owners, reward accounting per identity per epoch, one script to check it. A percentage cap per entity in FIP.02/FIP.05, which needs no part of this document. The allocation oracle, non-binding, as soon as the mapping is live, so that every depositor has a benchmark. Then the curve, two parameters in the reward script, switched on after two Songbird cycles and the split test in Appendix D. In parallel and independently, a FIP for contract-originated P-chain delegation. Scope B only behind that FIP or a reviewed key set with the custody matrix in Section 8 filled in; and if neither, an explicit decision that B is not coming.

The one thing this RFP asks the network not to do is to treat the Sceptre case as closed when the second identity is deregistered and the chill expires. The identity was never the problem and the chill never reached the pool. The 2.4 billion is still there, its ownership is still a matter of decoding transactions by hand, and the next operator to reach the ceiling will do the arithmetic Sceptre did, in the dark, unless the dark is removed first.

---

## Appendix A — Minimum Specification for the Settlement Bridge

Written from outside the codebase, as the questions a core reviewer would have to answer for each route. Where a mechanism is named, it is the Avalanche-derived mechanism Flare's `go-flare` inherits; where it is not, the RFP does not know.

**A.1 P-chain primitive (§5.2 ii).** A new P-chain transaction type, or an extension of `AddPermissionlessDelegatorTx`, whose reward owner and change owner are a C-chain contract address rather than a P-chain key, and which the P-chain accepts only when accompanied by a proof that the named contract emitted a matching intent (entity, nodes, amount, duration) in a finalised C-chain block. The proof can follow the pattern `PChainStakeMirror` already uses in the other direction. Intents are emitted only at reward-epoch boundaries, batched per epoch by the router, so the transaction count is bounded by the entity count, not by depositor activity; a contract cannot flood the P-chain because the contract only speaks once per epoch. C-chain reorgs are not a concern past finality, and the P-chain acts only on finalised intents. At maturity the P-chain returns principal and rewards to the contract's atomic-memory address, and the router imports it in the next epoch. What the core team would need to state: whether the atomic export/import path can be driven without a P-chain signer at all, whether the mirror's verifier set is the right trust base for the reverse proof, and what the Stage 3 consensus changes do to any of this.

**A.2 FCC enclave (§5.2 i).** A TEE image that holds the pool's P-chain key material (many addresses, three validators each, per the mirror limit in §5.2), generated inside the enclave, with remote attestation of the image hash published on-chain and checked by the eFLR contract before any instruction is honoured. The image executes one policy: read the router's target allocation from a finalised C-chain block, construct the export, delegate and import transactions that move toward it, and sign nothing else. The RFP would need the Foundation to state which TEE (the FCC roadmap will decide this, and the answer determines the side-channel history the design inherits), who can publish a new image hash and under what delay, and what happens to in-flight delegations when an image is retired: the honest answer is that they mature under the old key and the new image must be able to import them, which means the key has to be transferable between images or the retirement has to wait a full lock ladder. That constraint alone is an argument for A.1.

**A.3 Either route.** The router's target allocation is the only input the bridge accepts, the bridge's only outputs are delegations and imports, and both are visible on-chain every epoch. Anyone can recompute the allocation from public inputs and compare. A bridge whose behaviour cannot be recomputed by an outsider is a key, whatever it is called.

---

## Appendix B — Numbers, with dates

All figures below are the ones the argument leans on. Estimates are marked and their derivation is in "The Ceiling on Owning" (6 September 2026); this appendix does not recompute them.

| Figure | Value | Source, date |
|---|---|---|
| Active stake | 21.51bn FLR | Flare Metrics, 5 Sep 2026 |
| Active validators | 179 | Flare Metrics, 5 Sep 2026 |
| Free delegation space | 8.6bn FLR | Flare Metrics, 5 Sep 2026 |
| Largest entity (Bifrost) | 1.14bn FLR, four nodes under the 300M node cap | Flare Metrics, 5 Sep 2026 |
| Sceptre pool | 2.384bn FLR, 11.08% of active stake | Ceiling on Owning, 6 Sep 2026 |
| Entity ceiling | 1.2bn FLR (4 × 300M) | FIP.05, FIP.16 fork 14 Jul 2026 |
| Sceptre delegated share | 66.4% of pool | Ceiling on Owning (estimate) |
| Pool rewards, annualised | ~228M FLR/yr | Ceiling on Owning (estimate, epochs 426–428) |
| Rewards on delegated portion | ~151M FLR/yr | Ceiling on Owning (estimate) |
| 20% fee on delegated portion | ~30M FLR/yr | Ceiling on Owning (estimate) |
| Sceptre service fee (10%) | ~23M FLR/yr | Ceiling on Owning (estimate) |
| Sceptre advertised buffer | ~50M FLR | Sceptre app, pre-July 2026 |
| Capital moved to self-bonds | ~140M FLR, seven tranches | Forum proposal §1.3, 31 Aug 2026 |
| Gross pool yield "today" | 7–8% | Sceptre UI, September 2026; varies by page and date, treat as a range |
| Inflation | 3% | FIP.16 |
| Do-nothing baseline (§2.5) | 1.18bn excess × ~6.3% × 20% ≈ 15–19M FLR/yr in third-party fees | derived from the above |
| eFLR vs self-validating pool (§7.1) | 0.76g vs 0.78g net to depositor | derived |

| sFLR total supply | 1.196963bn sFLR | Flare explorer, contract 0x12e6…c2BB, 8 Sep 2026 (author-verified) |
| sFLR supply, peak | 1.339bn sFLR, 3 Jun 2026 | S. Hudspeth, daily on-chain totalSupply, 400 days (not independently re-run) |
| sFLR supply, change | −10.6% peak to 8 Sep; September −58.5M, the worst month in the series; 5 and 6 Sep among the ten largest burn days | same |
| sFLR pool, change since 31 Aug 2026 | −107M FLR in eight days; flat (±5M) for the three weeks before; largest single day 5 Sep; about a quarter to stFLR, the rest out of pooled staking | S. Hudspeth, on-chain, 9 Sep 2026 (not independently re-run; consistent with the share series at the implied rate) |
| sFLR exchange rate, implied | ~1.8–2.0 FLR per sFLR | CoinGecko sFLR $0.0122 / FLR ~$0.0066, 8 Sep 2026; reconciles 1.197bn shares with the 2.384bn FLR pool figure |

Bifrost is the live demonstration of the document's first claim: the node ceiling already touches entity concentration (four nodes at cap is 1.14bn) and touches the pool not at all.

Two notes on the sFLR rows. The supply series is in shares; the pool figure this document uses is in FLR, and because the exchange rate rises with rewards, a falling share count understates the outflow less than a falling FLR balance would overstate it. Sceptre's own dashboard labels an FLR-denominated line as "Total sFLR", which rises with rewards while shares leave; the share count is the clean number. And the decline began on 3 June, three months before the identity breach was public, so the breach accelerated an outflow it did not start.

## Appendix C — Liquidity under stress

A pool normalised to 1.0, a reserve of 5%, delegated capital laddered uniformly over L days so that 1/L of it matures daily, no re-delegation while the queue is non-empty, redemptions served from the reserve first and maturities second. Outflow arrives in one day or spread over thirty. The table gives days until the queue is empty.

| Ladder L | Outflow 5% / 1 day | 5% / 30 days | 10% / 1 day | 10% / 30 days | 20% / 1 day | 20% / 30 days |
|---|---|---|---|---|---|---|
| 90 days | 1 | 30 | 5 | 30 | 15 | 30 |
| 180 days | 1 | 30 | 10 | 30 | 29 | 36 |
| 365 days | 1 | 30 | 20 | 35 | 58 | 65 |

Readings. A 5% reserve absorbs a 5% one-day run and nothing larger. A 90-day ladder clears a 20% run in two weeks; a 365-day ladder takes two months, during which the token trades at whatever discount the secondary market sets, and every day of that discount is a day of pressure on the invariant. The reserve size and the ladder ceiling are the two parameters the Songbird run should set, from observed redemption behaviour, before the Flare deployment. The model omits secondary-market absorption (which shortens the queue) and reflexive exits triggered by the discount (which lengthen it); both should be added on Songbird data.

## Appendix D — Songbird test plan

"Two epochs under load" is a phrase, not a plan. Before the curve binds on Flare: eight to twelve reward epochs on Songbird with Scope 0 live and the oracle publishing, then four with the curve on; a split test, in which a test operator divides one entity into three and its attributed share under the mapping must not change; a rate check, in which the paid rate per operator matches f(s_i) to within rounding every epoch; a forced parameter change through the full timelock with the ragequit window exercised; a measured tracking error against the target every epoch, with the step and budget constraints of 6.3 visible in the ladder. Before Scope B on Flare, additionally: an artificial run of 10% and 20% of the pool against the reserve and queue, with observed secondary-market discount; an image rotation under route (i) through the custody path in Section 8, including a full ladder run-off under the old key; a mirror test with the pool's stake spread over more than three nodes per address to confirm the vault architecture accounts correctly; a pause by a single party and its expiry; and a deliberate desynchronisation between the reward script's view of the pool and the P-chain's, to see what the accounting does.

## Appendix E — Threat model

| Adversary | Reach | Bound by |
|---|---|---|
| TEE compromise (route i) | Extract keys, redirect maturing delegations, drain reserve | Nothing after the fact; only by choosing route (ii) or accepting the loss (11.13) |
| Poisoned host / AS data | Move correlation output | Correlation is advisory (6.8); no consensus effect |
| Governance capture of Y or filters | Steer delegation to a faction | 2-of-3 with community veto; timelock; ragequit; step limit (6.3) |
| Queue griefing | Drain reserve to floor, force discount, demand a patch | Immutable invariant; in-order queue; attacker exits at own discount (5.3) |
| Reward-script desync C/P | Pool paid on stale or wrong allocation | Tracking-error publication; fail-closed rule (11.9); Appendix D test |
| Identity splitting | Second entity, reset s_i | Scope 0 operator mapping; curve does not bind before it; Appendix D split test |
| Validator cartel on separate infrastructure | Aggregate share across friendly entities | Not bound here; identity-staking follow-up (11.3) |
| Foundation as sole key holder | Everything (Scope B) | Custody matrix (8); if unmet, no Scope B |

## Reviews

Draft 2 responded to a structured written review; Draft 3 to a second that found the Stage 3 gap and the missing enforcement point. Draft 4 responds to Steven Hudspeth (@hudspeth589) and Jon, operators on Flare who have built the LST kit and bond NFT described in Section 3, and whose objection that the router pays for compliant boxes is answered in 7.4 by agreeing with it. Draft 5 responds to their second round: the definitions in Section 0, Scope 0 in Section 4, the shared-ABI correction in Section 3, the moat objection and the documented-versus-inferred split in 7.4, the sFLR supply series in Appendix B, and Appendix F. Draft 6 responds to the third: the disclosed/undisclosed axis (§1, §2.3, §3), the curve in place of the router (§4, §6.10), the incidence of reward-based sanctions on delegators (§0, §4.1), and the sequencing evidence (§4, Appendix B). The router is theirs to have killed. Their interest is disclosed as the author's is: they build a competitor to sFLR and, above the threshold, to this proposal.

## Appendix F — What FIP.16 says, and what this document infers from it

Quotations are from proposals.flare.network/FIP/FIP_16.html (accepted 24 April 2026). Inferences are the author's.

| FIP.16 says | Where | This document infers | Status |
|---|---|---|---|
| "the economic incentives for infrastructure providers – and their delegators – will transition from FLR inflation to organic yield driven by on-chain activity" | §1.1 | Provider income is meant to come from activity fees, not from winning delegation | Inference; the sentence is about the source of rewards, not about how providers compete for them |
| FDC fees "distributed to infrastructure providers and their delegators"; FCC fees "distributed directly to entities and stakers" | §4.1, §4.3 | Feeds and attestations that have customers are the activity the network pays for | Direct reading |
| "the role of data providers will expand and become more and more prominent" | §4.4.1 | Data quality is the axis the network expects entities to compete on | Inference |
| Entities "have to run independent infrastructure and data acquisition services … This requires a level of professionalism and sufficient funding. To prevent a race to the bottom and dumping strategies, a minimum 20% entity fee will be applied network wide" | §5.2 | The floor is a funding mechanism for infrastructure, and a fee market above it was not the intention | Inference; §5.2 forbids a race to the bottom, it does not forbid a race to the top |
| "further incentivize staking over delegation … to increase locked supply and thus ensure economic stability for validators" | §1 | P-chain stake is the leg the network wants capital in; a pool's C-chain leg should be the minimum liquidity needs | Direct reading, applied in §6.9 |
| Nothing on what a validator should be, how one is funded, or whether marketing to stakers is wanted | — | This document's 7.4 argument that stakers are "the wrong capital" | Author's position, not the network's |


## Appendix G — The retired design: a router as a reward filter

Drafts 3 to 5 proposed to enforce the allocation oracle as a reward-eligibility rule above a threshold, with a schedule from 11% to 5%. It is kept here because the threshold schedule, the enforcement-point analysis and the three edges are still the correct account of what a cliff would take, and because the reasons it was retired (§4.1) are only legible against the thing itself.

**Scope A — Enshrined router, private tokens.** The protocol publishes a delegation allocation every reward epoch: a target set of entities and weights computed from the formula in Section 6, with per-entity caps and capacity. Private LSTs continue to exist, issue their own tokens and hold their own keys. Above a threshold share of network stake, a pool's delegation is reward-eligible only if its observed allocation is within a tolerance ε of the published target. That is the enforcement point, and it has to be named, because a router that pools "must use" is a forum mandate unless one of three edges exists: the P-chain accepts delegation only from allow-listed originators, the reward script pays pool delegation only when it matches the router, or the entity registry binds pool contracts. The second is the only one available without a consensus change and without custody, and it is what "Scope A" means from here: a reward filter, not an appeal. A pool may still export, self-bond and side-deal; it then forfeits staking and FSP rewards on the non-conforming portion, which is the one lever the network already holds over every delegator on it.

Below the threshold, Scope A touches nothing. Kits, bond NFTs, direct stakers, small pools, and every private LST under the line operate as they do today, with their own keys and their own selection. The router is not a plan for the network's stake; it is a rule for the layer at which the market has already produced one pool choosing a ninth of the validator set, and it is silent everywhere else. If twenty kit instances stay small, they never meet it. If one of them reaches the line, the same rule applies to it as to sFLR, and a rule that applied only to the pool that exists today would be a sanction, not a design.

The threshold is a political fact before it is a number. Sceptre is above any threshold this RFP would propose. Either the document says the largest pool is bound on day one, or the threshold is a schedule. The proposal is a schedule with a sunset: 11% at activation (binding nobody), stepping to 8% and then 5% at 25-epoch intervals, about three months each, so that the pool that exists today has two quarters to bring its allocation inside the target before its rewards depend on it. A schedule is a concession; it is also the difference between a rule Sceptre can comply with and one it has to fight.

Its failure was not enforcement, which the reward script could do, but measurement: it counted size and could not see whether the size was disclosed. Its second failure was that it needed an allocator, and an allocator that binds is a participant. The curve in §6.10 keeps the enforcement point (the reward script) and drops both.

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
