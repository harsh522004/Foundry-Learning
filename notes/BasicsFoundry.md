Foundry Overview: Forge & Cast Explained
========================================

Think of Foundry as a **Swiss Army knife for Solidity developers**. It's a toolchain --- a collection of individual tools that each do one job really well. Here's the full picture first:

| Tool | What it is | Analogy |
| --- | --- | --- |
| `foundryup` | Installer/updater | Like `nvm` for Node.js |
| `forge` | Build, test, deploy | Like `npm` + `jest` + `hardhat` |
| `cast` | Interact with blockchain | Like `curl` but for Ethereum |
| `anvil` | Local blockchain node | Like Ganache / Hardhat Network |
| `chisel` | Solidity REPL | Like Python's interactive shell |

* * * * *

FORGE --- The Build & Test Engine
-------------------------------

Forge is what you use **during development**. Every time you write code, compile, test, or deploy --- that's Forge.

### What Forge does:

**1\. Compiling your contract**

`forge build`

This compiles all your `.sol` files and puts the output (ABI + bytecode) into the `out/` folder.

**2\. Running tests**

`forge test`

Your test files live in `test/`. Forge finds any function starting with `test` and runs it. It's blazing fast because it runs natively in Rust --- much faster than Hardhat/Truffle.

```
// Example test file
contract MyTest is Test {
    function testAddition() public {
        assertEq(1 + 1, 2);
    }
}

```

**3\. Deploying contracts**

`forge script script/Deploy.s.sol --rpc-url $RPC_URL --broadcast`

You write a "Script" (a special Solidity file) that describes your deployment, and Forge executes it. This is why the course feels "strict" --- instead of just running a JS deploy script, you write deployment logic in Solidity itself. This is actually powerful because you test the same language you deploy in.

**4\. Installing dependencies**

`forge install OpenZeppelin/openzeppelin-contracts`

It uses Git submodules instead of npm. Takes getting used to.

**5\. Gas snapshot**

`forge snapshot`

Shows you exactly how much gas each function costs --- super useful before going to mainnet.

* * * * *

CAST --- Your CLI Blockchain Swiss Army Knife
-------------------------------------------

This is where people get confused. Cast is essentially **a command-line tool to talk to any EVM blockchain** --- read data, send transactions, do conversions, encode/decode calldata --- without writing any code.

Think of it as: **"anything you'd do in Etherscan or a script, you can do in one terminal command."**

### Cast has 3 main categories of commands:

* * * * *

### Category 1: READ from the blockchain (no gas, no wallet needed)

```
# Get ETH balance of an address
cast balance 0xYourAddress --rpc-url $RPC_URL

# Call a read-only function on a contract
cast call 0xContractAddress "functionName()" --rpc-url $RPC_URL

# Example: read the "totalSupply" of a token
cast call 0xTokenAddress "totalSupply()" --rpc-url $RPC_URL

# Get the latest block number
cast block-number --rpc-url $RPC_URL

```

* * * * *

### Category 2: WRITE to the blockchain (needs wallet + gas)

```
# Send ETH
cast send 0xRecipient --value 0.1ether --private-key $PK --rpc-url $RPC_URL

# Call a write function on a contract
cast send 0xContractAddress "store(uint256)" 42 --private-key $PK --rpc-url $RPC_URL

```

* * * * *

### Category 3: UTILITY / Conversions (no network needed at all)

This is where Cast really shines. Blockchain has a lot of number/encoding weirdness, and Cast handles it:

```
`# Convert Wei to Ether
cast to-unit 1000000000000000000 ether
# Output: 1.000000000000000000

# Convert Ether to Wei
cast to-wei 1 ether
# Output: 1000000000000000000

# Convert hex to decimal
cast to-dec 0xFF
# Output: 255

# Convert decimal to hex
cast to-hex 255
# Output: 0xff

# Get function selector (first 4 bytes of keccak256)
cast sig "transfer(address,uint256)"
# Output: 0xa9059cbb

# Decode calldata
cast calldata-decode "transfer(address,uint256)" 0xa9059cbb...

```

* * * * *

### Why the course uses Cast heavily --- The Mental Model

The course uses Cast because in production, you often need to **quickly verify** something on-chain without spinning up a whole frontend. For example:

> "Did my deployment actually work? Let me just `cast call` the contract right now."

Here's how Forge + Cast work together in a typical workflow:

```
1\. Write contract (.sol)        → Forge compiles it
2. Write tests (.t.sol)         → Forge runs them
3. Write deploy script (.s.sol) → Forge broadcasts it
4. Verify deployment            → Cast call to check state
5. Interact manually            → Cast send to call functions

```

* * * * *

ANVIL --- Your Local Blockchain
-----------------------------

`anvil`

Starts a local Ethereum node instantly. Gives you 10 funded test accounts with private keys printed in the terminal. Forge tests run against this by default. It's like Ganache but much faster.

* * * * *