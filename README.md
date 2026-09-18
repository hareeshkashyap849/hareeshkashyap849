# Wuzilin

I build Ethereum smart contracts and the TypeScript services that index and display
them. The four repositories here are one thread: an ERC-4626 vault on Base Sepolia, the
indexer and query API behind it, two front ends over it, and a separate Uniswap V3 swap
indexer on Base mainnet that tests how far the same approach scales.

## Where to look

| Target role | Repository | The one thing to verify | Live |
|---|---|---|---|
| Solidity / smart contracts (junior–mid) | [`erc4626-vault`](https://github.com/wuzilin-web3/erc4626-vault) | `forge test` → 46 passed, 0 failed, 12 fork tests skipped without an RPC endpoint. The invariant suite was checked for teeth: reversing one rounding direction in `convertToAssets` turns three tests red. | [dApp](https://wuzilin-web3.github.io/erc4626-vault/) |
| Web3 backend / indexer | [`erc4626-vault-dapp`](https://github.com/wuzilin-web3/erc4626-vault-dapp) | Open the committed `data/vault.sqlite` with `node:sqlite`: it starts at block 46,919,124 on chain 84532 — one block *before* the vault's own deployment block, 46,919,125, which is allowed and says so in the README — and holds the one real `Deposit`, at block 46,919,498. No runtime dependencies, no `npm install`. | not hosted |
| Web3 full-stack dApp | [`vault-console`](https://github.com/wuzilin-web3/vault-console) | 259 tests across 10 files, green with nothing else running. The published `/history` page labels its figures as a snapshot rather than live, because a static host runs no process. | [console](https://wuzilin-web3.github.io/vault-console/) |
| Chain data at scale | [`base-swap-indexer`](https://github.com/wuzilin-web3/base-swap-indexer) | Run the backfill, don't open a database: that repository commits its code and its recorded measurement — 96,980 swaps over a 200,000-block Base mainnet window, reproduced with `npm run index -- --blocks 200000` — and no database file, because `data/` is ignored. The RPC ceilings it ran into are in that README, which also says plainly why the backfill's duration is not recorded anywhere. | not hosted |

## Public and verifiable

- **Two front ends, both published.** The vault's no-build vanilla-JS dApp at
  <https://wuzilin-web3.github.io/erc4626-vault/>, and the Next.js console at
  <https://wuzilin-web3.github.io/vault-console/>.
- **Contract:** [`0x7941438ee07bea4469ccd4bec583e9fb24037f35`](https://sepolia.basescan.org/address/0x7941438ee07bea4469ccd4bec583e9fb24037f35)
  on Base Sepolia (84532), deployed at block 46,919,125 — the block containing the deployment
  transaction — and funded with 21 USDC of test assets. Testnet only: no real funds,
  not audited.
- **Index measurements — one read out of a committed database, one recorded in the README that produced
  it.** The committed one is `erc4626-vault-dapp/data/vault.sqlite`: open it and the vault index begins
  one block before its own deployment block (46,919,124 against 46,919,125, which is safe in that
  direction and is why the snapshot's start is not asserted equal to it) and holds the real deposit.
  The other is `base-swap-indexer`'s 96,980 swaps over the 200,000-block window 51,192,412 to 51,392,411
  on Base mainnet — a recorded measurement a reader reproduces by running the backfill, because that
  repository commits no database. It no longer quotes a duration for it either: the 37 minutes it once
  stated beside this window is not in the database it examines.
- **Tests:** 46 Foundry tests, plus 12 fork tests against the real mainnet USDC contract
  and 259 front-end tests. Slither runs 102 detectors over 18 contracts and reports
  nothing in the project's own code: the 32 findings a full run reports are all in the
  vendored OpenZeppelin library. Medusa runs the same 9 properties and reports 0
  failures.

## What is not there yet

- **No hosted long-lived service.** The vault index is a committed SQLite snapshot kept
  current by a scheduled workflow rather than a resident server, because no free tier
  keeps a process alive. `erc4626-vault-dapp` states which parts of that are proven and
  which are still a plan.
- **No Postgres, no queue, no message broker.** One chain and one contract fit in SQLite;
  adding infrastructure before there is a load to justify it would be decoration.
- **One project domain.** All four repositories are an ERC-4626 vault and a single
  Uniswap V3 pool: depth in one thread, not breadth. Nothing here runs against mainnet
  with real funds.
