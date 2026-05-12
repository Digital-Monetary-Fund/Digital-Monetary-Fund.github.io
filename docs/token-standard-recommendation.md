# Token Standard Recommendation — DMF Stablecoin Portfolio

**Status:** Draft for review
**Scope:** 205 ISO-4217 fiat-pegged stablecoins, deployed on Ethereum + BSC, with Escrow and other smart functions required.
**Date:** 2026-05-12

---

## Recommendation

Use **ERC-20 + EIP-2612 (`permit`) + optional ERC-1363 (`transferAndCall`)** as the base for all 205 currencies. Implement **Escrow as a separate companion contract**, not baked into the token.

Reserve **ERC-3643 (T-REX)** as a per-currency exception for any jurisdiction that mandates KYC-gated holders. Do not unify the whole portfolio on it.

---

## Why ERC-20 wins on reach

- Every CEX, DEX, wallet, custodian, bridge, oracle, and accounting tool speaks ERC-20.
- BSC's BEP-20 is ERC-20-compatible — same source compiles to both chains DMF currently targets, plus Polygon, Arbitrum, Base, Avalanche if expanded later.
- Universally accepted = the dominant consideration for tokens that need to behave like fiat across the widest possible audience.

## Why not the alternatives

The standards that *look* more capable for Escrow each have a disqualifying problem:

| Standard | MetaMask | Cold wallets (Ledger/Trezor) | DeFi (Uniswap, Aave, Curve…) | CEXs |
|---|---|---|---|---|
| **ERC-777** | Shows as ERC-20; sends work, hooks invisible | Generic ERC-20 signing; no awareness of hooks | **Hostile.** Several majors explicitly block 777 after the imBTC/Lendf.me reentrancy exploit (~$25M, 2020). Limited integrations even where allowed. | Mostly fine for listing (read as ERC-20), but some operationally avoided it post-imBTC |
| **ERC-1155** | Fungible 1155 balances render as second-class — NFT-style UI, not a currency row in the asset list | Ledger Live treats 1155 as NFTs; fungible balances usually not surfaced. Sending requires dApp UI, not the wallet itself | **Effectively zero.** Uniswap/Aave/Curve/Compound are all ERC-20-only. You'd be locked out of the major liquidity venues | Very few CEXs ingest fungible 1155 — deposit/withdraw plumbing is built around ERC-20 |
| **ERC-3643** | Renders as ERC-20 (it is, at the interface level). Approvals identical | ERC-20-shaped calls — no special support needed | **Blocked by design.** Transfers require both sides whitelisted. AMM pools and lending markets are contracts; they'd need bespoke whitelisting per protocol. Used on permissioned security-token venues (Tokeny, INX, Securitize), not Uniswap | Listed on regulated security-token exchanges/ATSs only. Mainstream CEXs don't list 3643 as currencies |

**Net:** wallet and signing support is the easy part — 777 and 3643 both wear an ERC-20 interface. The real fractures are downstream:

- ERC-777 dies in DeFi due to its reentrancy history.
- ERC-1155 dies in both DeFi and CEX — the fungible-currency tooling stack assumes ERC-20.
- ERC-3643 dies in permissionless DeFi by design — the whitelist *is* the point.

## Recommended technical stack

### Base contract
- **ERC-20** using OpenZeppelin's audited implementation.
- 18 decimals internally; bankers' rounding applied at the UX layer (matches the `/currencies/` page description).
- Mint/burn for supply adjustment per the existing asset-backed model.

### Extensions
- **EIP-2612 `permit`** — gasless approvals via signed message. Turns Escrow's `approve → deposit` into a single transaction. Cheap to add, supported by every modern wallet.
- **ERC-1363 `transferAndCall`** (optional, alongside not instead of standard ERC-20) — lets a single transfer atomically trigger the Escrow contract for counterparties that support it. Falls back gracefully to standard ERC-20 flows otherwise.

### Companion contracts
- **Escrow** as a separate contract that holds the ERC-20 tokens. Keeps the token contract boring and maximally compatible, lets Escrow logic be upgraded without touching 205 currency contracts, and avoids "transfer has surprising side effects" — the exact thing that gets tokens delisted.
- Other smart functions (e.g. scheduled payments, conditional transfers, multi-sig releases) follow the same pattern: separate contracts that interact via standard ERC-20 / permit / transferAndCall.

### Deployment pattern
- **Factory contract** for the 205 deployments. One audited template, per-currency parameters: name, symbol, ISO 4217 code, mint authority, initial supply.
- Same template on Ethereum and BSC.

## Tradeoffs to be aware of

- **Two-step Escrow flow:** plain ERC-20 forces `approve + deposit`. EIP-2612 closes this gap with no compatibility cost; ERC-1363 closes it more elegantly when counterparties support it. Together they make the UX competitive with ERC-777 without the risk.
- **No on-chain compliance hooks in plain ERC-20.** If any specific jurisdiction (e.g. CNY, RUB, certain emerging market currencies) requires KYC for the underlying fiat, deploy *that* currency as ERC-3643 from the factory's permissioned template — don't compromise the rest of the portfolio.
- **DeFi liquidity is contract-mediated.** Anything that requires hooks, callbacks, or whitelists at the token level will be excluded from Uniswap-class venues. Keep the token contract minimal; put intelligence in companion contracts that hold the tokens.

## Open questions for review

1. Which jurisdictions, if any, will require permissioned (ERC-3643) variants on day one?
2. Should the factory support upgradeability (proxy pattern) or fixed deployments? Upgradeable means faster bugfix response but introduces governance trust assumptions.
3. Cross-chain strategy: native deployments on each chain vs. bridged from a canonical chain? Affects how mint/burn authority is held.
4. Audit scope: re-audit the new template + Escrow + Factory together, or rely on existing Coinscope/CoinTool/OpenZeppelin coverage of the original contract?
