# Contracts

The on-chain layer lives in [P2Flux/contracts](https://github.com/P2Flux/contracts), together with
its ABIs, EIP-712 definitions and the tests that pin them to the deployed bytecode.

| contract | what it does |
|---|---|
| `P2FluxSplitter` | one-time payments — settles a payment and splits the fee in a single transaction; the buyer sends it and pays gas in ETH |
| `P2FluxRecurring` | recurring payments — charges a customer-signed EIP-712 authorization once per period |
| `P2FluxSponsoredSplitter` | one-time payments where the buyer pays the network fee in USDC and holds no ETH — the buyer signs an EIP-3009 authorization, the P2Flux relayer sends it |
| `P2FluxGasSponsor` | subscription signup, allowance restore and allowance removal for a buyer holding no ETH — an EIP-2612 permit plus an EIP-3009 fee authorization, sent by the relayer |

All four are immutable once deployed: not upgradeable, not proxied, no `delegatecall`, no owner
function that can move funds. None holds a balance between calls, so there is nothing in any of them
for anyone — including P2Flux — to withdraw. The two sponsored contracts accept calls only from the
relayer, and a relayer can move nothing a customer did not sign for: every fee is bounded by the
signed amount and by an immutable hard cap of 0.25 USDC per sponsored operation.

## Addresses

Base Mainnet (8453), all Sourcify exact match:

| contract | address |
|---|---|
| USDC (Circle, native) | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| `P2FluxSplitter` | `0x5A3bD0945cd0C80B124870881dE49a717D20E0D0` |
| `P2FluxRecurring` | `0xb415A9910Ef627e3bEF10F5Cb9DC92a3271e0975` |
| `P2FluxSponsoredSplitter` | `0x95E18ec05D4282acB3aab7aD60325bA4EEeEa8df` |
| `P2FluxGasSponsor` | `0xD1DDAaa301403d18fD4A23Fc69493ef48af90285` |

Base Sepolia (84532), test environment:

| contract | address |
|---|---|
| USDC | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` |
| `P2FluxSplitter` | `0x3120aa022437db5c6d0439ac7f7852ce8b38e70f` |
| `P2FluxRecurring` | `0x394c3fe285168f333ebf29e8f3585039328f2a73` |
| `P2FluxSponsoredSplitter` | `0x876f7b98e8c06291ec916a3223a92038b0a8774f` |
| `P2FluxGasSponsor` | `0x2dc51643040d7c396f1199a0664ac095d4b89ec5` |

`GET /v1/capabilities` returns the same addresses per operation (`sponsor_contracts`); read them from
the API rather than pinning constants, which is also what keeps an integration correct across
environments.

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
deploying a new contract. One-time: 1% (`ONE_TIME_BPS`), on both splitters. Recurring: 2% plus a
0.10 USDC network fee per collection, both merchant-funded out of the amount. A sponsored one-time
payment additionally carries a merchant-funded fixed network fee of 0.10 USDC
(`FIXED_NETWORK_FEE`) and the buyer's quoted network fee, capped by `MAX_NETWORK_FEE_HARD_CAP`
(0.25 USDC). The current values are in
[`src/recurring.ts`](https://github.com/P2Flux/contracts/blob/main/src/recurring.ts) and
[`src/sponsored.ts`](https://github.com/P2Flux/contracts/blob/main/src/sponsored.ts).

## Using the ABIs

```bash
npm install github:P2Flux/contracts
```

```ts
import { recurringAbi, recurringTypedData } from '@p2flux/contracts/recurring'
import { splitterAbi, paymentIdFor } from '@p2flux/contracts/splitter'
import { sponsoredSplitterAbi, gasSponsorAbi } from '@p2flux/contracts/sponsored'
import { BASE_MAINNET, BASE_SEPOLIA } from '@p2flux/contracts/addresses'
```

Or take the raw JSON from `abi/` in that repository.
