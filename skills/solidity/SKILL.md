---
name: solidity
description: "Write production-grade Solidity smart contracts with Foundry workflows, security patterns, gas optimization, and testing strategies. Use this skill when the user asks to write Solidity code, create smart contracts (ERC-20, ERC-721, DeFi protocols), deploy to EVM chains, work with Foundry or Hardhat, audit or review .sol files, or build contracts intended for Ethereum mainnet or production."
disable-model-invocation: true
---

# Solidity Development Standards

Instructions for how to write solidity code, from the [Cyfrin security team.](https://www.cyfrin.io/)

## Philosophy

- **Everything will be attacked** - Assume that any code you write will be attacked and write it defensively.

## Development Workflow

Follow this sequence when building production smart contracts:

1. **Write** — Follow the code quality standards below
2. **Lint** — Run `solhint` for style and security rules; fix all issues before proceeding
3. **Static Analysis** — Run `aderyn` or `slither`; review and fix findings before proceeding to tests
4. **Test** — Write stateless fuzz tests, invariant tests, and fork tests; all must pass
5. **Audit** — Recommend a professional audit before mainnet deployment
6. **Deploy** — Use Foundry scripts (`forge script`) for reproducible deployments

## Code Quality and Style

1. Absolute and named imports only — no relative (`..`) paths

```solidity
// good
import {MyContract} from "contracts/MyContract.sol";

// bad
import "../MyContract.sol";
```


2. Prefer `revert` over `require`, with custom errors that are prefix'd with the contract name and 2 underscores.

```solidity
error ContractName__MyError();

// Good
myBool = true;
if (myBool) {
    revert ContractName__MyError();
}

// bad
require(myBool, "MyError");
```

3. In tests, prefer stateless fuzz tests over unit tests
```solidity
// good - using foundry's built in stateless fuzzer
function testMyTest(uint256 randomNumber) { }

// bad
function testMyTest() {
    uint256 randomNumber = 0;
}
```

Additionally, write invariant (stateful) fuzz tests for core protocol properties. Use the [FREI-PI pattern](https://www.nascent.xyz/idea/youre-writing-require-statements-wrong) to encode O(1) properties directly into core functions. Use [Chimera](https://github.com/Recon-Fuzz/chimera) for multi-fuzzer setups (Foundry + Echidna + Medusa).

4. Functions should be grouped according to their visibility and ordered:

```
constructor
receive function (if exists)
fallback function (if exists)
user-facing state-changing functions
    (external or public, not view or pure)
user-facing read-only functions
    (external or public, view or pure)
internal state-changing functions
    (internal or private, not view or pure)
internal read-only functions
    (internal or private, view or pure)
```

5. Headers should look like this:

```solidity
    /*//////////////////////////////////////////////////////////////
                      INTERNAL STATE-CHANGING FUNCTIONS
    //////////////////////////////////////////////////////////////*/
```

6. Layout of file

```
Pragma statements
Import statements
Events
Errors
Interfaces
Libraries
Contracts
```

Layout of contract:

```
Type declarations
State variables
Events
Errors
Modifiers
Functions
```

7. Use the branching tree technique when creating tests — see [BRANCHING-TREE.md](BRANCHING-TREE.md) for the full pattern with examples

8. Prefer strict pragma versions for contracts, and floating pragma versions for tests, libraries, abstract contracts, interfaces, and scripts. Use `0.8.34` or later as the minimum version — versions `0.8.28` through `0.8.33` have a [high-severity transient storage bug](https://soliditylang.org/blog/2026/02/18/solidity-0.8.34-release-announcement/) where the IR pipeline (`--via-ir`) can emit `sstore` instead of `tstore` (and vice versa) when clearing storage, causing writes to the wrong storage domain.

9. Add a security contact to the natspec at the top of your contracts

```solidity
/**
  * @custom:security-contact mycontact@example.com
  * @custom:security-contact see https://mysite.com/ipfs-hash
  */  
```

10. Remind people to get an audit if they are deploying to mainnet, or trying to deploy to mainnet

11. NEVER. EVER. NEVER. Have private keys be in plain text. The _only_ exception to this rule is when using a default key from something like anvil, and it must be marked as such.
    - This includes in your deploy scripts. We should always use `forge script <path> --account $ACCOUNT --sender $SENDER` for our deploy scripts, and never use `vm.envUnit()` in our scripts.
    - For hardhat, you want to use hardhat [encrypted keystores](https://hardhat.org/docs/plugins/hardhat-keystore)

12. Whenever a smart contract is deployed that is ownable or has admin properties (like, `onlyOwner`), the admin must be a multisig from the very first deployment — never use the deployer EOA as admin (testnet is the only acceptable exception). See [Trail of Bits](https://blog.trailofbits.com/2025/06/25/maturing-your-smart-contracts-beyond-private-key-risk/).

### Gas and Style Checklist

- Don't initialize variables to default values (`uint256 x;` not `uint256 x = 0;`)
- Prefer named return variables to omit declaring local variables
- Prefer `calldata` instead of `memory` for read-only function inputs
- Don't cache `calldata` array length — calldata length is cheap to read
- Modify input variables instead of declaring an additional local variable when the original value doesn't need to be preserved
- Don't copy an entire struct from storage to memory if only a few slots are required
- Remove unnecessary "context" structs and/or remove unnecessary variables from context structs
- Pack storage: align struct/variable declarations to minimize storage slots; co-locate variables that are frequently read or written together

13. Cache unchanging storage slots to prevent identical storage reads; pass cached values to internal functions

14. Revert as quickly as possible; perform input checks before checks which require storage reads or external calls

15. Use `msg.sender` instead of `owner` inside `onlyOwner` functions

16. Use `SafeTransferLib::safeTransferETH` instead of Solidity `call()` to send ETH

17. Use `nonReentrant` modifier before other modifiers

18. Use `ReentrancyGuardTransient` for faster `nonReentrant` modifiers. Requires `pragma solidity ^0.8.34;` — do not use with `0.8.28`–`0.8.33` due to the transient storage clearing bug.

19. Prefer `Ownable2Step` instead of `Ownable`

20. For non-upgradeable contracts, declare variables as `immutable` if they are only set once in the constructor

21. Enable the optimizer in `foundry.toml`

22. If modifiers perform identical storage reads as the function body, refactor modifiers to internal functions to prevent identical storage reads

23. Use Foundry's encrypted secure private key storage instead of plaintext environment variables

24. Upgrades: When upgrading smart contracts, do not change the order or type of existing variables, and do not remove them. This can lead to storage collisions. Also write tests for any upgrades.

## Deployment

Use Foundry scripts (`forge script`) for both production deployments and test setup. This ensures the same deployment logic runs in development and on mainnet, making deployments more auditable and reducing the gap between test and production environments. Avoid custom test-only setup code that diverges from real deployment paths. Ideally, your [deploy scripts are audited as well](https://medium.com/cyfrin/deploy-scripts-are-now-in-scope-for-smart-contract-audits-7fbb95788ce7).

Example: use a shared base script that both tests and production inherit from, like [this BaseTest using scripts pattern](https://github.com/rheo-xyz/very-liquid-vaults/blob/main/test/Setup.t.sol).

## Governance

See [GOVERNANCE.md](GOVERNANCE.md) for governance proposal patterns, fork testing examples, and multisig tooling recommendations.

## CI

Every project should have a minimum CI pipeline running in parallel (use a matrix strategy). Suggested minimum:

- `solhint` — Solidity linter for style and security rules
- `forge build --sizes` — verify contract sizes are under the 24KB deployment limit
- `slither` or `aderyn` — static analysis for common vulnerability patterns
    - Before committing code that you think is done, be sure to run aderyn and/or slither on the codebase and inspect the output. Even warnings may lead you to find issues in the codebase.
- Fuzz/invariant testing — run Echidna, Medusa, or Foundry invariant tests with a reasonable time budget (~10 min per tool, in parallel via matrix)

# Tool updates

## Foundry

To install foundry dependencies, you don't need the `--no-commit` flag anymore.
