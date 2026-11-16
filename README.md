# GIWA Launchpad — Ammora Season 2 Builder mission

A working token launchpad web app built with `@ammora-protocol/sdk@0.3.9` on
**GIWA Sepolia testnet** (chain 91342). Testnet only — no real money, no mainnet.

## What it does

- **Lists live launches** — discovers `LaunchCreated` events from the Ammora
  auto-route LaunchFactory (`0x35353a2Dab7426EcCAe6f5FB267702a542ab68fA`) via the
  Blockscout logs API (RPC `eth_getLogs` in 10k-block chunks as fallback), then
  reads each token's name/symbol/decimals and each curve's live state with the
  SDK (`AmmoraV4LaunchCurveClient.getStateAtBlock`,
  `getMigrationProgressAtBlock`). Works fully read-only, no wallet needed.
- **Trade on a curve** — buy with WETH / sell tokens on any Active curve.
  Quotes come from `quoteExactInputAtBlock`; orders are planned with
  `planSwapExactInput` (1% slippage, 20-min deadline), exact-amount WETH
  approval first when required, then the swap is sent through the user's own
  connected wallet after a `publicClient.call` preflight simulation.
- **Graduate a curve** — one-click permissionless migration for curves in
  `ReadyToMigrate` status (`planMigrateAtBlock`).
- **Create a Config** — derives a new immutable Config from reviewed template
  #6 (verified on-chain: enabled, hash matches the guide), with the user's own
  recipient addresses, sale supply and graduation threshold. Preview shows the
  full economics + review hash before signing.
- **Create a launch** — issues a new token + bonding curve on any Config
  (`buildCreateAmmoraV4Launch`), parses the receipt for the token/curve
  addresses, and adds it to the list.
- **Wrap ETH → WETH** — buys settle in WETH; the wallet panel wraps on demand.

## Run locally

```bash
npm install
npm run build        # esbuild -> dist/app.js
npx serve .          # or: python3 -m http.server 8931
# open http://localhost:8931
```

No build step is needed to edit: `src/` is plain JS + the SDK + viem.

## Deploy as a static site

`index.html`, `styles.css` and `dist/app.js` are all that is needed — no server,
no environment variables, no secrets. Any static host works (Cloudflare Pages,
Vercel, Netlify, GitHub Pages, IPFS). Example with Cloudflare Pages:

```bash
npx wrangler pages deploy . --project-name giwa-launchpad
```

## Tests (all read-only, run on the live GIWA Sepolia RPC)

```bash
npm run build                 # bundle must pass
node test-discover2.mjs       # 73 launches found, live curve states read
node test-quote.mjs           # quote 0.001 WETH -> tokens, price math check
node test-template6.mjs       # core verification + template #6 hash check
node test-batch.mjs           # batched transport: 19-field state = 1 HTTP req
node test-e2e.mjs             # full pass in headless Chromium (needs the
                              # test proxy: node test-proxy.mjs &)
node test-smoke.mjs           # (jsdom; see BUILD_NOTES.md — jsdom cannot run
                              #  <script type="module">, so this is limited)
```

## Files

- `src/config.js` — pinned deployment addresses (auto-route V4, GIWA Sepolia)
- `src/helpers.js` — `deriveConfigFromTemplate` port + price/format helpers
- `src/app.js` — the whole app (wallet, discovery, trade, create flows)
- `index.html`, `styles.css`, `build.mjs`
- `test-*.mjs` — test evidence scripts (see BUILD_NOTES.md)
- `BUILD_NOTES.md` — full findings, deploy instructions, risks/limits
