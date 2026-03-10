📨 Transaction Types --- Simply Explained
=======================================

First, What Even IS a Transaction "Type"?
-----------------------------------------

Every transaction you send to a blockchain is a **packet of data**. Over the years, Ethereum improved *how that packet is structured* --- like upgrading an envelope format. Each new format got a "type number."

That's all transaction types are --- **different envelope formats for sending data to the blockchain.**

* * * * *

The Evolution: Why New Types Were Needed
----------------------------------------

Think of it like this --- Ethereum started with one format, and as problems were discovered, they upgraded it:

### Type 0 --- `0x0` --- "Legacy" (the original, ~2015)

The very first transaction format. Simple and straightforward.

```
{
  nonce, gasPrice, gasLimit,
  to, value, data, v, r, s
}

```

The problem with it: you set one `gasPrice` (e.g. "I'll pay 50 gwei"). But Ethereum's network demand fluctuates wildly. You'd either **overpay** when the network was quiet, or get **stuck** in a queue when the network was busy because your price was too low.

* * * * *

### Type 1 --- `0x1` --- "Access List" (EIP-2930, 2021)

Added one new field: an **access list** --- a pre-declared list of storage slots your transaction will touch. This made certain complex contract interactions slightly cheaper on gas. Rarely used in practice, mostly an intermediate step.

* * * * *

### Type 2 --- `0x2` --- "EIP-1559" (2021) ← **Current Default on Ethereum**

This was the big one. Ethereum introduced a smarter gas fee mechanism:

```
Instead of:  gasPrice (you guess one number)

Now you set:
  maxFeePerGas        → "I'll pay at most X gwei total"
  maxPriorityFeePerGas → "I'll tip the validator Y gwei"

```

The network itself sets a **base fee** per block (which gets burned 🔥). You pay `baseFee + tip`. You never overpay beyond your `maxFeePerGas`.

This is why today when you send ETH in MetaMask, you see "Base fee" and "Priority fee" --- that's Type 2 in action.

**This is what Foundry uses by default** when you deploy to Anvil or Ethereum.

* * * * *

### Type 113 --- `0x71` --- ZkSync's Own Type (EIP-712)

ZkSync invented its own transaction type for features that don't exist on Ethereum --- most importantly, **Account Abstraction**.

Normal Ethereum: only an EOA (Externally Owned Account --- your MetaMask wallet) can initiate transactions. Smart contracts can't initiate, they can only react.

With Type 113 / Account Abstraction: **smart contracts can act as wallets**. Your wallet can have custom logic like "require 2-of-3 signatures" or "pay gas in USDC instead of ETH." This is a ZkSync superpower.

`Type 113 = ZkSync's native format, enables smart wallet features`

* * * * *

The Full Picture in One Table
-----------------------------

| Type | Hex | Name | Key Feature | Default on |
| --- | --- | --- | --- | --- |
| 0 | `0x0` | Legacy | Simple, single gasPrice | Old Ethereum, ZkSync local |
| 1 | `0x1` | Access List | Pre-declare storage slots | Rarely used |
| 2 | `0x2` | EIP-1559 | baseFee + tip model | **Ethereum / Anvil** |
| 113 | `0x71` | EIP-712 | Account abstraction | **ZkSync native** |

* * * * *

Why You See `0x2` vs `0x0` in the `/broadcast` Folder
-----------------------------------------------------

When Foundry deploys your contract, it logs everything in `/broadcast/run-latest.json`. The `type` field in that file tells you which envelope format was used.

```
Deploy to Anvil (normal)      → type: "0x2"  (EIP-1559, modern default)
Deploy to ZkSync local node   → type: "0x0"  (Legacy, because --legacy flag)

```

ZkSync's local node (`anvil-zksync`) doesn't fully support EIP-1559 style transactions yet, so you use `--legacy` to force Type 0, which it understands. Without that flag, the deployment would fail or behave unexpectedly.

* * * * *

The `-legacy` Flag in Plain English
-----------------------------------

`forge create MyContract --zksync --legacy`

The `--legacy` flag is just you saying: **"Package this transaction in the old Type 0 format instead of the modern Type 2 format."**

ZkSync's local test node expects this older format. On ZkSync's actual mainnet/testnet, it'll eventually use Type 113 natively --- but for local testing, `--legacy` is the practical workaround the course teaches.