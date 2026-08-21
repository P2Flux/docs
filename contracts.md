# Contracts

The on-chain layer lives in [P2Flux/contracts](https://github.com/P2Flux/contracts), together with
its ABIs, EIP-712 definitions and the tests that pin them to the deployed bytecode.

| contract | what it does |
|---|---|
| `P2FluxSplitter` | one-time payments — settles a payment and splits the fee in a single transaction |
| `P2FluxRecurring` | recurring payments — charges a customer-signed EIP-712 authorization once per period |

Both are immutable once deployed: not upgradeable, not proxied, no `delegatecall`, no owner function
that can move funds. Neither holds a balance between calls, so there is nothing in either contract
for anyone — including P2Flux — to withdraw.

The one mutable value is `P2FluxRecurring.relayer`, changeable only by an `admin` address that is
itself immutable. That exists so a compromised relayer hot key can be rotated without redeploying
and without touching a single customer authorization.

## What the customer signs

An authorization names the payer, the recipient, the token, the amount, the period, a start and an
end, and a salt. The contract enforces exactly those terms: the amount cannot be raised, the period
cannot be shortened, and the recipient cannot be changed after signing. A charge outside the signed
window, or a second charge inside one period, reverts.

## Fees

Fees are contract constants, not configuration — readable on chain, and unchangeable without
deploying a new contract. The current values are in
[`src/recurring.ts`](https://github.com/P2Flux/contracts/blob/main/src/recurring.ts).

## Using the ABIs

```bash
npm install github:P2Flux/contracts#v0.2.0
```

```ts
import { recurringAbi, recurringTypedData } from '@p2flux/contracts/recurring'
import { splitterAbi, paymentIdFor } from '@p2flux/contracts/splitter'
```

Or take the raw JSON from `abi/` in that repository.
