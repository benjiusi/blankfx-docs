# Swaps

## What's new

- **Pre-connect indicative rate** — see an oracle-based estimated rate before connecting your wallet
- **Fee disclosure on quote step** — the fee is shown as `0.05% · $X.XX` before you confirm
- **TradFi terminology** — familiar language: "Get FX Rate", "Confirm Exchange", "Rate tolerance", "Exchange settings"
- **Gasless relay** — swap without holding ETH for gas; Permit2 + relay router handle submission

---

## How pricing works

BlankFX uses oracle-anchored pricing for all FX swaps. Instead of deriving the rate from pool ratios (like a standard AMM), the protocol reads the current FX rate from on-chain oracles and applies a small spread.

This means:

- The rate you see is close to the interbank mid-rate
- Large trades don't cause significant price impact
- There's no MEV extraction opportunity on your trade

## Swap types

The protocol routes your trade automatically based on the tokens involved:

| From | To | Route |
|------|-----|-------|
| USDC | USDT | USD Hub (direct swap between USD stablecoins) |
| USDC | EURC | Cross-currency (USD Hub → EUR Pool via oracle rate) |
| EURC | USDC | Cross-currency (EUR Pool → USD Hub via oracle rate) |
| USDT | USDC | USD Hub (direct swap between USD stablecoins) |

You don't need to think about routing. The interface selects the optimal path automatically.

## USD Hub swaps

Swaps between USD-denominated stablecoins (USDC, USDT, RLUSD) happen inside the USD Hub. These are essentially 1:1 swaps with a minimal spread, since all tokens represent the same underlying currency.

## Cross-currency swaps

When you swap between different currencies (e.g. USDC to EURC), the protocol:

1. Takes your input token into the USD Hub
2. Reads the EUR/USD oracle rate
3. Calculates the output amount minus spread
4. Sends the output token from the EUR Pool to your wallet

Settlement is atomic — either the full swap completes or nothing happens. No partial fills.

## Gasless swaps

BlankFX supports gasless execution. Instead of submitting an on-chain transaction yourself (which requires ETH for gas), you sign an off-chain message and the protocol's relay service submits it for you.

The gas cost is built into the swap spread, so you never see a separate gas charge.

**Requirements for gasless swaps:**
- One-time token approval to the Permit2 contract (this is an on-chain transaction that costs gas)
- After that, all swaps are gasless via the BlankFXRelayRouter

## Slippage and minimum output

Every swap includes a minimum output amount to protect you from unexpected rate changes between quoting and execution. The default rate tolerance is 0.5%.

If the rate moves unfavorably beyond your tolerance, the transaction reverts and you keep your tokens. You can adjust this in **Exchange settings**.
