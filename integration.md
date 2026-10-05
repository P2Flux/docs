# Integration guide

**This guide now lives at [p2flux.com/docs](https://p2flux.com/docs/)**, where it is generated from
the implementation and cannot drift from it.

Start there:

| | |
|---|---|
| [Quick start](https://p2flux.com/docs/quickstart.html) | one USDC payment, end to end |
| [Environments](https://p2flux.com/docs/networks.html) | Base Sepolia (test) and Base Mainnet (production, live) — addresses, assets, confirmation policy |
| [Payments](https://p2flux.com/docs/payments.html) | one-time payments, `gas_payment_mode` (network fee paid in USDC, no ETH required), verification, lost-callback recovery |
| [Subscriptions](https://p2flux.com/docs/subscriptions.html) | authorization, charging, cancellation |
| [Refunds](https://p2flux.com/docs/refunds.html) | merchant→buyer transfers, and the one-refund rule |
| [AI agent payments](https://p2flux.com/docs/agents.html) | AI agents pay per request over x402: the P2Flux facilitator, the paywall calls, prepaid, fees |
| [Agent Paywall](https://p2flux.com/docs/agent-paywall.html) | the WordPress plugin that charges AI agents per page |
| [Errors](https://p2flux.com/docs/errors.html) | every result code and what to do about it |
| [SDKs](https://p2flux.com/docs/sdks.html) | the official clients: `composer require p2flux/sdk-php`, `npm install @p2flux/sdk`, `composer require p2flux/laravel` |
| [API reference](https://p2flux.com/docs/api/) | the interactive contract, from [`openapi.json`](openapi.json) |

The version of this file that used to live here described eight SDK calls and thirteen result codes.
The API has fifteen endpoints and fifty-one codes, and it had never mentioned refunds, payment
recovery or the hosted checkout. Rather than half-correct it, it is replaced by documentation
generated from the code — and by `openapi.json`, which the API's own test suite checks against the
routes on every change.

## x402 facilitator, without an SDK

AI agents pay over x402. P2Flux is the facilitator: point any x402 v2 resource server (the official
`@x402/*` middleware, for example) at it, and it checks and settles the agent's payment on Base.

| | Facilitator URL | Network |
|---|---|---|
| Test | `https://api-test.p2flux.com/x402` | `eip155:84532` (Base Sepolia) |
| Production | `https://api.p2flux.com/x402` | `eip155:8453` (Base Mainnet) |

| Endpoint | |
|---|---|
| `GET /x402/supported` | What the facilitator settles: `exact` and `upto` on the network above, and the `eip2612GasSponsoring` extension. Your middleware reads it at start-up. |
| `POST /x402/verify` | Body `{ x402Version, paymentPayload, paymentRequirements }`. Checks the payment without sending anything. `200` with `isValid`, and `invalidReason` when false. |
| `POST /x402/settle` | Same body. Settles the payment; P2Flux's relayer pays the gas. `200` with `success`, `transaction`, `network`, `payer`, and `errorReason` when false. |
| `POST /x402/vault` | Body `{ recipient }`, your wallet. Returns `pay_to`, your vault, and the `extra` for your route config. Fetch it once: it never changes for a wallet. |

- `payTo` must be your vault, and `extra.p2flux.recipient` your wallet. The vault can only pay your
  wallet, less the P2Flux fee; P2Flux never holds funds.
- A payment that cannot settle is a `200` with a reason, not an HTTP error. Only a malformed request
  (`400`), a rate limit or a server failure is an HTTP error.
- A payment is reported settled once. Sent again, it gets `errorReason: "invalid_transaction_state"`:
  serve nothing.
- `errorReason: "settlement_pending"` carries the hash of a transaction still being mined. Repeat the
  identical settle call once; it is never broadcast twice.
- Fee: 1% of each settlement, at least 0.003 USDC. Smallest payment 0.01 USDC. There is no free tier.
- Prepaid balances (x402 batch-settlement, 3%) are not listed in `/supported`. They are offered
  through the paywall challenge, `POST /x402/paywall/challenge`. A paywall price may go down to
  0.0001 USDC: it is offered at that price from a prepaid balance and at 0.01 paid per request.

A seller without an x402 library uses the paywall calls instead (`/x402/paywall/challenge`,
`/x402/paywall/redeem`, and `/x402/paywall/verify` for usage pricing), or the SDK helpers that wrap
them. Every field is in [`openapi.json`](openapi.json); the walk-through is
[AI agent payments](https://p2flux.com/docs/agents.html).
