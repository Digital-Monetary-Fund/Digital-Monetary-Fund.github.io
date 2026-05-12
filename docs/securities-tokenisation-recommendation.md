# Securities Tokenisation Recommendation — Tokenised Shares & Equity

**Status:** Draft for review
**Scope:** Tokenisation of securities (equity shares, fund units, debt instruments) — distinct from the DMF stablecoin portfolio.
**Date:** 2026-05-12
**Companion to:** `token-standard-recommendation.md` (stablecoins)

---

## Recommendation

Use **ERC-3643 (T-REX)** paired with **ONCHAINID** for tokenised equity. Pair with a jurisdiction whose company law recognises the token register as the authoritative share register (Switzerland DLT Act, Liechtenstein TVTG, Luxembourg Blockchain Law III, or Delaware DGCL §224).

Keep **ERC-1400** in mind as the conceptual reference standard — its partition model is worth understanding for issuers with multiple share classes in one vehicle.

---

## Why securities need a different standard from stablecoins

For the DMF stablecoin portfolio we argued *against* ERC-3643 because its whitelist breaks permissionless DeFi. For securities, **that same whitelist is the feature**. Shares aren't supposed to trade on Uniswap; they trade on regulated venues (Archax, INX, ADDX, SIX SDX, tZERO) where both sides of every trade are KYC'd. The standard's "limitation" is the regulator's requirement.

Securities have requirements stablecoins don't:

- **Identity-gated holders** — only KYC'd, accredited, jurisdictionally eligible wallets can hold or receive.
- **Forced transfers** — courts, regulators, lost-key recovery. The issuer must be able to move shares without the holder's key.
- **Corporate actions** — dividends, splits, voting, rights issues.
- **Lockups & vesting** — Reg D 6-month, Reg S 12-month, founder vesting schedules, employee option cliffs.
- **Share classes** — common vs preferred, voting vs non-voting, restricted vs free-trading. Same issuer, different rights.
- **Document binding** — prospectus, shareholder agreements, terms anchored to the token.

Plain ERC-20 provides none of these. ERC-20 + off-chain compliance is what most early STOs shipped — it doesn't scale and doesn't survive regulatory scrutiny.

---

## ERC-3643 vs ERC-1400

| | **ERC-3643 (T-REX)** | **ERC-1400** |
|---|---|---|
| **Origin** | Tokeny, 2021+, actively maintained | Polymath et al., 2018–19, largely frozen |
| **Identity** | ONCHAINID — separate identity contract per holder, reusable across issuances | None native; bolted on per implementation |
| **Compliance** | Modular compliance contract — pluggable rules (jurisdiction, max holders, lockups) | Reason-coded transfer restrictions (ERC-1594) |
| **Forced transfers** | Built-in via agent role | ERC-1644 controller operations |
| **Share classes** | One token per class (clean) | Partitions (multiple "tranches" in one contract — elegant but heavier) |
| **Wallet UX** | ERC-20 interface — shows in MetaMask, Ledger | ERC-20 interface — same |
| **Live institutional use** | Tokeny, Archax (UK FCA), INX, several EU-licensed STO platforms | Largely historical; conceptual influence remains |

ERC-3643 wins on **adoption that matters for securities**: regulated STO venues recognise it, EU MiCA/DLT-pilot platforms accept it, and ONCHAINID gives you portable KYC — a holder verified once for issuer A is reusable for issuer B without re-KYC.

ERC-1400's partition model is genuinely better for issuers with many share classes in one vehicle (e.g. fund tokens with different fee tranches). For straightforward equity, ERC-3643's "one token per class" is simpler.

---

## Recommended technical stack

### Identity layer
- **ONCHAINID** — a per-holder identity contract that holds verifiable claims (KYC status, accreditation, jurisdiction, sanctions checks). Issued and signed by accredited claim issuers.
- Reusable across issuances, so a holder isn't re-KYC'd for every new security.

### Token contract
- **ERC-3643** template. One contract per share class.
- Agent role for forced transfers (court orders, lost-key recovery).
- Compliance contract attached — modular rules per jurisdiction.

### Compliance modules
Common building blocks to compose:
- Country / jurisdiction allowlist
- Maximum holders (e.g. 99 for some private placements)
- Per-holder maximum balance
- Lockup periods (Reg D 6-month, Reg S 12-month, founder vesting cliffs)
- Investor accreditation requirements
- Daily / monthly transfer limits

### Corporate actions
- **Dividends** — pull-payment pattern (preferred) or push distribution; both with on-chain proof of holdings at snapshot block.
- **Voting** — snapshot-based, off-chain signature (e.g. Snapshot.org pattern) with on-chain settlement for binding votes.
- **Splits / consolidations** — implemented as a balance recalculation event with the agent role.
- **Rights issues** — separate ERC-3643 token for the rights, exchangeable for new shares.

### Document binding
- IPFS-pinned prospectus, shareholder agreement, terms — hash stored on-chain.
- ERC-1400 patterns (ERC-1643 document standard) can be adopted alongside ERC-3643 even though it's a separate lineage.

---

## What the standard does *not* solve

- **Legal wrapper.** The token represents legal equity only if the issuer's articles, subscription agreement, and (in most jurisdictions) a registrar/transfer agent recognise it as such. Pick a jurisdiction whose company law lets a token be the authoritative share register:
  - **Switzerland** — DLT Act (2021)
  - **Liechtenstein** — TVTG (Token and TT Service Provider Act, 2020)
  - **Luxembourg** — Blockchain Law III (2021)
  - **Delaware** — DGCL §224 (amended 2017 to allow blockchain share registers)
  - **France** — PACTE law / Ordonnance 2017-1674
- **Venue.** Decide upfront which regulated trading venue(s) will list. They'll often dictate the standard — Tokeny-aligned issuers default to ERC-3643; SIX SDX uses CMTA's framework.
- **Cap table reconciliation.** Carta, Pulley etc. don't natively read security tokens. Either bridge data via an oracle/agent or accept on-chain as authoritative.
- **Settlement finality.** For listed-venue trading, atomic delivery-vs-payment usually requires a stablecoin or tokenised cash leg on the same chain — relevant link to the DMF stablecoin stack.

---

## How this relates to the DMF stablecoin portfolio

The two stacks intentionally diverge:

| | **Stablecoins (DMF currencies)** | **Securities (shares)** |
|---|---|---|
| **Base standard** | ERC-20 + EIP-2612 + optional ERC-1363 | ERC-3643 + ONCHAINID |
| **Holder restrictions** | None — permissionless | Whitelist via ONCHAINID claims |
| **DeFi pools** | Required (liquidity is the point) | Excluded by design |
| **Venues** | CEXs, DEXs, any wallet | Regulated STO venues only |
| **Forced transfer** | No — would get tokens delisted | Yes — required by regulators |
| **Document anchoring** | Not needed | Prospectus, terms hashed on-chain |
| **Legal jurisdiction** | DAO-stateless | Specific jurisdiction whose law recognises token registers |

Both stacks can settle against each other on the same chain — a DMF stablecoin is a natural cash leg for delivery-vs-payment on a tokenised-share trade.

---

## Open questions for review

1. Which jurisdiction will DMF (or a sister entity) incorporate the issuer under for any tokenised-equity offerings? This is the single most consequential decision and constrains everything downstream.
2. Will tokenised equity be issued by DMF itself (DAO governance tokens with legal share status) or by separate issuer entities using DMF rails?
3. Which regulated venue(s) — Archax, INX, ADDX, SIX SDX, tZERO, or a new direct listing — define the target standard?
4. Is ONCHAINID acceptable as the identity layer, or does a specific venue/jurisdiction require an alternative (e.g. Tokeny vs Securitize DS Protocol vs CMTA)?
5. Are there share-class structures that would benefit from ERC-1400 partitions over per-class ERC-3643 tokens?
