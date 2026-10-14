<!-- solve-problem:9338a347-ba01-408d-ae7c-438d29d0df04 -->
# Auditable full-cost searcher profitability

A [Tangle Solve problem](https://solve.tangle.tools/problems/9338a347-ba01-408d-ae7c-438d29d0df04).

## What is the problem?

The task is to build an **auditable, full-cost accounting of MEV searcher profitability** — i.e., to measure the *true economic profit* of MEV searchers (agents competing to capture arbitrage, liquidation, and sandwich value on DEXes and across CEX–DEX venues), not just their gross extracted value.

Formal setting: for each searcher (or searcher bundle) $i$ over a time window $T$, gross profit is observable-ish on-chain as $\text{token flow} - \text{gas} - \text{builder/validator payments}$. The problem asks to estimate

$$\pi_i = \underbrace{V_i^{\text{MEV}}}_{\text{extracted value}} - \underbrace{C_i^{\text{gas}} - C_i^{\text{bids}}}_{\text{on-chain}} - \underbrace{C_i^{\text{CEX fees, inventory/financing}} - C_i^{\text{failed txs/bids}} - C_i^{\text{infra, colocation, staffing}} - C_i^{\text{private transfers}}}_{\text{off-chain / unobserved}}$$

where several cost terms (and some revenue legs, e.g., CEX fills with hidden identities, private payment channels, subsidies from exchanges/market makers) are **not directly observable**. The requirement is *auditable*: full-cost estimates must come with rigorous **bounds derived from partial identification** — explicit assumptions under which upper/lower profit bounds hold — rather than point estimates from unvalidated heuristics.

The key epistemic constraint: distinguishing genuine searcher efficiency from (a) omitted costs and (b) implicit subsidies (e.g., zero-fee CEX maker programs, exchange rebates, vertically-integrated financing). This is an experimental/empirical problem: the deliverable is a methodology plus measurement on real data, with reproducible bound construction.

## Why does it matter?

Current MEV literature systematically overstates searcher profits by counting gross extracted value while ignoring CEX trading fees, inventory financing costs, infrastructure (colocation, engineers), failed bids, and private transfers. If measured profits are inflated, we wrongly conclude MEV extraction is an attractively competitive market, misjudge the degree of centralization pressure among searchers/builders, and miscalibrate policy (e.g., order-flow auctions, PBS design) on whether MEV is efficiently competed away. A rigorous full-cost methodology with partial-identification bounds would also transfer to adjacent auditing problems — any setting where on-chain evidence is partial and off-chain costs must be bounded.

## What's already known?

The anchor work is Wu et al. (2025), arXiv:2507.13023, which performs the most extensive accounting to date: it links CEX and DEX trades across venues, reconstructs searcher strategies (notably CEX–DEX arbitrage), and estimates profitability. However, its accounting is *partially inferred*: CEX legs are matched probabilistically (exchange API data or assumed behavioral signatures rather than confirmed account-level trades), and several cost categories are estimated with heuristic assumptions rather than formal bounds. The census record explicitly flags this as the gap.

Prior MEV measurement work (Daian et al. 2019 on chain arbitrage; Qin et al. 2022 on MEV-Boost builder/payment data; Flash Boys 2.0-adjacent sandwich measurements) largely measures gross extracted value minus on-chain costs only; off-chain costs are acknowledged but excluded. The adjacent methodological literature the problem points to: **partial identification / bounds methods in econometrics** (Manski-style set identification) and **privacy-preserving financial auditing** (zero-knowledge proofs of solvency, exchange proof-of-reserves) — neither has been applied to searcher profit accounting. No published work provides confidence bounds on searcher profit under explicitly stated identification assumptions. That is the open territory.

## What would count as solving it?

An experimental paper/methodology that:

1. **Formalizes the observables and assumptions.** A stated data-generating model: what is on-chain-observable, what is CEX-API-observable, what is unobservable; explicit assumptions on unobservables (e.g., fee schedules, financing rates bounded by market rates, infrastructure cost lower bounds from observable telemetry).
2. **Derives profit bounds.** For each searcher cohort and time period, an interval $[\underline{\pi}_i, \overline{\pi}_i]$ valid under the stated assumptions — a partial-identification result, not a point estimate.
3. **Validates auditability.** Either (a) ground-truth validation against a cooperative searcher's real books, or (b) sensitivity analysis showing how bounds tighten/loosen with each assumption, plus reproducible open-source pipeline over public chain and market data.
4. **Answers the economic question.** Under full-cost accounting, are searcher profits consistent with near-competitive rents, or is there persistent excess profit attributable to subsidies/omissions in prior studies?

A referee accepts: rigorous bound derivation + empirical application at scale + reproducibility. A bonus would be ZK-assisted auditing where searchers provably reveal cost data without disclosure.

## How to start

1. **Read Wu et al. (2025) first**, especially its profitability accounting sections and appendix assumptions — catalogue exactly which cost terms are inferred vs. observed, and where identification is weakest (likely CEX leg matching and infrastructure costs). Build the discrepancy table; that table is your research agenda.
2. **Import the partial-identification toolkit**: Manski-style bounds; interval arithmetic over per-term bounds. Start simple — bound each cost category independently and compose intervals; sensitivity analysis per term tells you which observation would tighten the bound most (an information-value ranking).
3. **Deal with attribution**: searcher identity clustering is itself error-prone (EOAs, contract wallets, builder relationships). Bound misclassification error or work at the cohort level first.
4. **Main obstacle**: data access — CEX account-level trades are private; you must either partner with a searcher/exchange for a validation cohort, or lean on public CEX market-data + fee schedules to bound trading costs without account access.

## Key references

- Wu, K., et al. (2025). *Measuring CEX-DEX Extracted Value and Searcher Profitability: The Darkest of the MEV Dark Forest*. arXiv:2507.13023. The anchor study: reconstructs CEX–DEX arbitrage strategies and searcher accounting at scale, with partially inferred CEX legs and cost categories — the baseline to extend. https://arxiv.org/abs/2507.13023
- Daian, P., et al. (2019). *Flash Boys 2.0: Frontrunning, Transaction Reordering, and Consensus Instability in Decentralized Exchanges*. arXiv:1904.05234. Established on-chain arbitrage/liquidation measurement (gross, on-chain costs only) — the methodology being generalized. https://arxiv.org/abs/1904.05234
- Qin, K., Zhou, L., Gervais, A. (2022). *Quantifying Blockchain Extractable Value: How dark is the forest?* IEEE S&P 2022, arXiv:2101.05511. Systematic gross-EV measurement across DeFi; defines the measurement baseline full-cost accounting must correct. https://arxiv.org/abs/2101.05511
- Manski, C. F. (2003). *Partial Identification of Probability Distributions*. Springer. The canonical treatment of set identification under weak assumptions — the methodological template for profit bounds. (Verify edition/details.)
- Daian et al. / proof-of-reserves ZK literature (e.g., Provisions-style exchange solvency proofs, 2014–2021) — privacy-preserving auditing primitives for the optional cooperative/ZK variant. (Verify specific citation before citing.)

## Warnings

- **Do not reproduce Wu et al.'s inferred CEX-match rate as ground truth** — their searcher–exchange account linkage is heuristic; bounds must propagate matching uncertainty.
- **Gross-EV numbers ≠ profit.** The single most common error in this literature is reporting extracted value minus gas as "profit."
- **Failed bids are systematically undercounted**: off-chain MEV-Boost bids that lose auctions leave no on-chain trace beyond bid-stream data, which requires private data-sharing agreements or telemetry assumptions.
- **Subsidy identification is the hardest part**: zero-fee maker programs and internal financing are invisible without cooperation; be explicit that bounds may be vacuous for vertically-integrated searchers absent assumptions.
- **Census-sourced caveat**: the record is thin; the "2025 CEX–DEX study" framing and status come from a secondary census entry — verify the arXiv record's actual scope before building on it.

---
*`BCGT-000324` · flagged for review*

## Acceptance guidance

An experimental paper/methodology that:

1. **Formalizes the observables and assumptions.** A stated data-generating model: what is on-chain-observable, what is CEX-API-observable, what is unobservable; explicit assumptions on unobservables (e.g., fee schedules, financing rates bounded by market rates, infrastructure cost lower bounds from observable telemetry).
2. **Derives profit bounds.** For each searcher cohort and time period, an interval $[\underline{\pi}_i, \overline{\pi}_i]$ valid under the stated assumptions — a partial-identification result, not a point estimate.
3. **Validates auditability.** Either (a) ground-truth validation against a cooperative searcher's real books, or (b) sensitivity analysis showing how bounds tighten/loosen with each assumption, plus reproducible open-source pipeline over public chain and market data.
4. **Answers the economic question.** Under full-cost accounting, are searcher profits consistent with near-competitive rents, or is there persistent excess profit attributable to subsidies/omissions in prior studies?

A referee accepts: rigorous bound derivation + empirical application at scale + reproducibility. A bonus would be ZK-assisted auditing where searchers provably reveal cost data without disclosure.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/9338a347-ba01-408d-ae7c-438d29d0df04?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000324.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000324)
