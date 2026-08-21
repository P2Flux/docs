# Integration guide

**This guide now lives at [p2flux.com/docs](https://p2flux.com/docs/)**, where it is generated from
the implementation and cannot drift from it.

Start there:

| | |
|---|---|
| [Quick start](https://p2flux.com/docs/quickstart.html) | one USDC payment, end to end |
| [Environments](https://p2flux.com/docs/networks.html) | Base Sepolia today, Base Mainnet pre-launch |
| [Payments](https://p2flux.com/docs/payments.html) | one-time payments, verification, lost-callback recovery |
| [Subscriptions](https://p2flux.com/docs/subscriptions.html) | authorization, charging, cancellation |
| [Refunds](https://p2flux.com/docs/refunds.html) | merchant→buyer transfers, and the one-refund rule |
| [Errors](https://p2flux.com/docs/errors.html) | every result code and what to do about it |
| [SDKs](https://p2flux.com/docs/sdks.html) | JS and PHP |
| [API reference](https://p2flux.com/docs/api/) | the interactive contract, from [`openapi.json`](openapi.json) |

The version of this file that used to live here described eight SDK calls and thirteen result codes.
The API has fifteen endpoints and fifty-one codes, and it had never mentioned refunds, payment
recovery or the hosted checkout. Rather than half-correct it, it is replaced by documentation
generated from the code — and by `openapi.json`, which the API's own test suite checks against the
routes on every change.
