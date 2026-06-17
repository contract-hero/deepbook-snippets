# DeepBook Integration Snippets

> Runnable, **sandbox-tested** reference apps for the most common
> [DeepBook V3](https://docs.sui.io/standards/deepbookv3) integration patterns on Sui.

**Landing page → https://contract-hero.github.io/deepbook-snippets/**

Each snippet is a small, self-contained mini-app (a headless core + a minimal Vite UI +
a live test suite) that runs end-to-end against the local
[deepbook-sandbox](https://github.com/MystenLabs/deepbook-sandbox). The framing throughout:
**DeepBook is infrastructure, not a product** — every snippet is a team choosing *not* to
rebuild the matching engine, and spending its effort on the product on top.

These snippets are the reference apps behind the
[`acc-deepbook-course`](https://github.com/contract-hero/acc-deepbook-course) — a hands-on
DeepBook course for the Agentic Community College framework. Here they live on their own so
you can clone, run, and crib from them without the course harness.

## The Core 4

| Pattern | Snippet | Key SDK / Move calls | Teaching moment |
|---|---|---|---|
| **A** — swap over a CLOB | [`01-swap`](./01-swap) | `deepBook.swapExactQuoteForBase` (no BalanceManager) | the `minOut` slippage guard; the `[base, quote, deep]` return tuple |
| **B** — orders on Spot | [`02-orders`](./02-orders) | `balanceManager.createAndShareBalanceManager` / `depositIntoManager`, `deepBook.placeLimitOrder` / `placeMarketOrder` / `cancelAllOrders` | the BalanceManager lifecycle; settled-vs-owed reconciliation |
| **E** — flash loan | [`03-flash-loan`](./03-flash-loan) | `flashLoans.borrowBaseAsset` + a Move module calling `pool::return_flashloan_base` | the `FlashLoan` hot-potato — repay in the same PTB or the whole tx reverts (LPs bear no risk) |
| **F** — market making | [`04-market-maker`](./04-market-maker) | `deepBook.placeLimitOrder` grid (POST_ONLY), `governance.stake` | two-sided quoting; one BalanceManager spans all pools; stake DEEP for rebates |

`01-swap`, `02-orders`, `04-market-maker` are TypeScript-only. `03-flash-loan` additionally
ships a Move package (`contracts/`) whose module consumes the hot-potato on-chain.

## Architecture (all snippets)

```
<snippet>/
├── src/sandbox.ts    load deployment manifest → SuiGrpcClient.$extend(deepbook(...)) + faucet
├── src/<core>.ts     the teachable surface — the SDK calls each pattern dissects (pure: takes client+signer)
├── src/App.tsx       minimal Vite UI that drives <core> (smoke driver)
├── tests/*.live.test.ts   end-to-end against the running sandbox
└── README.md         pattern summary, key calls, run steps
```

The connection helper `src/sandbox.ts` is copied into each snippet (self-contained, so each
runs standalone). It is adapted from the sandbox's own `examples/sandbox/setup.ts`.
`02-orders` and `04-market-maker` carry a `setupWithBalanceManagerBrowser` variant (for their
BalanceManager-backed UIs) that `01-swap`/`03-flash-loan` don't need — that's the only
intentional difference between the copies.

## Running against the sandbox

**1. Bring up the sandbox** (Docker required):
```bash
git clone https://github.com/MystenLabs/deepbook-sandbox
cd deepbook-sandbox/sandbox
pnpm deploy-all --quick      # wait for "DeepBook Sandbox Ready!"
```
This deploys DeepBook to a local Sui network, seeds `DEEP_SUI` + `SUI_USDC` pools with a
market maker, and writes `deployments/localnet.json`.

**2. Point the snippets at the deployment manifest.** By default `sandbox.ts` reads
`~/workspace/deepbook-sandbox/sandbox/deployments/localnet.json`. If your sandbox lives
elsewhere, override the path:
```bash
export DEEPBOOK_SANDBOX_DEPLOYMENTS=/path/to/deepbook-sandbox/sandbox/deployments/localnet.json
```

**3. Run a snippet** (live tests + UI):
```bash
cd 01-swap
pnpm install
pnpm test       # live — runs end-to-end against the sandbox (Docker must be up)
pnpm dev        # minimal UI on its own port (5181–5184)
```

The live tests fund a fresh ephemeral keypair from the sandbox faucet per case, so they're
isolated and never depend on prior on-chain state.

### `03-flash-loan` has an extra step

Its Move package must be published to the sandbox before `pnpm test`:
```bash
# DEEPBOOK_SANDBOX_DIR defaults to ~/workspace/deepbook-sandbox/sandbox
DEEPBOOK_SANDBOX_DIR=/path/to/deepbook-sandbox/sandbox \
  bash 03-flash-loan/contracts/scripts/deploy.sh
```
This publishes `arb_executor` and writes `deployment.json` with its package id. That id is
**chain-specific** — re-run `deploy.sh` after any sandbox redeploy. The Move package's
`Move.toml` dependency paths also point at the sandbox `.external-packages/` — edit them to
match your checkout.

## Links

- **Course** — [`contract-hero/acc-deepbook-course`](https://github.com/contract-hero/acc-deepbook-course)
- **Reference sandbox** — [`MystenLabs/deepbook-sandbox`](https://github.com/MystenLabs/deepbook-sandbox)
- **DeepBook V3 docs** — https://docs.sui.io/standards/deepbookv3
