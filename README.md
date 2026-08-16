# P2Flux documentation

Developer documentation for integrating P2Flux — programmable, non-custodial payments on Base.

| | |
|---|---|
| [Integration guide](integration.md) | the SDKs, the result table, and how a merchant drives a renewal |
| [Contracts](contracts.md) | what the on-chain layer does and where it lives |

## What P2Flux is

Payments move **buyer wallet → recipient wallet**, with the P2Flux fee split out in the same
transaction. There is no custody, no balances and no accounts: P2Flux never holds funds and never
becomes an intermediary that could.

Recurring payments work the same way. The customer signs an EIP-712 authorization once with an
ordinary EVM wallet, on terms they can read — amount, period, recipient, an end date. After that a
merchant's own system triggers each renewal, and the contract enforces one charge per period against
the terms that were signed. P2Flux has no scheduler and no database; **your application owns the
subscription lifecycle**, P2Flux executes the payment when you say so.

Cancellation belongs to the customer. P2Flux cannot revoke a wallet's authority, so the SDKs return
unsigned calldata for the customer's own wallet to send.

## Repositories

| repository | what |
|---|---|
| [contracts](https://github.com/P2Flux/contracts) | Solidity, ABIs, EIP-712 definitions, chain constants |
| [sdk-js](https://github.com/P2Flux/sdk-js) | JavaScript/TypeScript client |
| [sdk-php](https://github.com/P2Flux/sdk-php) | PHP client |

## Status

Base Sepolia. Not yet deployed to Base mainnet, and the SDKs are not yet published to npm or
Packagist — install them from a pinned git tag.

There is no OpenAPI specification yet. The authoritative description of every call and every result
code is the [integration guide](integration.md); a machine-readable spec is planned and will land
here when it is real rather than approximated.
