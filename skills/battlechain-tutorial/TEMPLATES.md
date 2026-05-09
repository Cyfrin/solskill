# BattleChain Deployment — Script Templates & Constants

This file contains the Solidity templates, constants, and generation patterns for Phase 3 script generation. Substitute all `[PLACEHOLDERS]` with real values collected during the question flow. Never leave a placeholder unfilled.

## BattleChain Testnet Constants (always use these)

```
BATTLECHAIN_CHAIN_ID  = 627
BATTLECHAIN_DEPLOYER  = 0x74269804941119554460956f16Fe82Fbe4B90448
AGREEMENT_FACTORY     = 0x2BEe2970f10FDc2aeA28662Bb6f6a501278eBd46
SAFE_HARBOR_REGISTRY  = 0x0A652e265336a0296816ac4D8400880E3e537c24
ATTACK_REGISTRY       = 0xdD029a6374095EEb4c47a2364Ce1D0f47f007350
BATTLECHAIN_CAIP2     = "eip155:627"
```

---

## Modifying Existing Deployment Scripts (Chain ID Branching)

**Do NOT create a separate `Setup.s.sol`.** Modify the project's existing deployment script(s) in `script/` to add chain-specific code paths using `block.chainid`. Existing mainnet/testnet logic must remain untouched.

### BCScript Inheritance

All generated scripts inherit `BCScript` from `cyfrin/battlechain-lib`, which provides:

- `bcDeployCreate()` / `bcDeployCreate2()` / `bcDeployCreate3()` -- deploy via BattleChainDeployer on BattleChain, or via CreateX (0xba5Ed...) on all other supported chains
- `defaultAgreementDetails()` -- auto-selects BattleChain scope + URI on BattleChain, or current chain's CAIP-2 scope + generic Safe Harbor V3 URI on other chains
- `createAndAdoptAgreement()` -- works on any chain with Safe Harbor registry/factory
- `requestAttackMode()` -- BattleChain only (reverts on other chains)
- `_isBattleChain()` -- runtime chain detection

### Chain ID Branching Pattern

Add a chain ID check in the `run()` function (or equivalent entry point):

```solidity
if (_isBattleChain()) {
    _deployBattleChain();
} else if (block.chainid == TARGET_L2_CHAIN_ID) {
    _deployL2();
} else {
    _deployDefault();
}
```

If the existing script doesn't already use helper functions, refactor the existing deployment logic into a `_deployDefault()` (or similar) internal function, then add the other functions alongside it. **Do not alter the behavior of the original path.**

### `_deployBattleChain()` Function Requirements

- Deploy contracts via `bcDeployCreate2(salt, bytecode)` (uses BattleChainDeployer -- CreateX + auto AttackRegistry registration)
- Use the same constructor arguments as the original path
- Swap external dependency addresses to their BattleChain equivalents (from Question 2), or deploy mocks if user chose that
- Replicate all post-deployment initialization calls from the original path (e.g. `setVault()`, `transferOwnership()`, `grantRole()`, `initialize()`)
- Add seeding logic at the end with the user's specified seed amount
- Log all deployed addresses with instructions to copy into `.env`

### `_deployL2()` Function Requirements (if user chose "Another L2" or "Both")

- If user chose CreateX: deploy via `bcDeployCreate2(salt, bytecode)` (calls CreateX directly on non-BattleChain chains)
- If user chose standard deployment: deploy with `new` or the project's existing pattern
- Swap external dependency addresses to their L2 equivalents
- Replicate all post-deployment initialization calls
- Log all deployed addresses

If the project has multiple deployment scripts (e.g. separate deploy + init scripts), add chain ID branching to each one as appropriate.

---

## New Script: `CreateAgreement.s.sol`

Create the Safe Harbor agreement with all user-specified terms. This script works on both BattleChain and non-BattleChain chains.

### Template Structure

- Inherit `BCScript` from `cyfrin/battlechain-lib`
- Populate `Contact[]` from the user's security contact info
- Use `defaultAgreementDetails()` which auto-selects:
  - **BattleChain:** `buildBattleChainScope` + `BATTLECHAIN_SAFE_HARBOR_URI`
  - **Other chains:** `buildChainScope` with runtime CAIP-2 + `SAFE_HARBOR_V3_URI`
- If the user needs custom scopes per chain, use `buildAgreementDetails()` with explicit `BcChain[]` and URI instead
- Populate `BountyTerms` with: bounty %, cap, retainable, identity, diligence, aggregate cap
- Set `agreementURI` to user's value or `""` if blank
- Call `createAndAdoptAgreement(details, deployer, salt)` (handles create + 14-day commitment + adopt)
- For non-BattleChain chains: the user must call `_setBcAddresses(registry, factory, attackRegistry, deployer)` to provide the Safe Harbor contract addresses on that chain
- Log `AGREEMENT_ADDRESS` for `.env`

---

## New Script: `RequestAttackMode.s.sol`

Submit the attack mode request. **BattleChain only.**

### Template Structure

- Guard with `require(_isBattleChain(), "Attack mode is BattleChain-only")`
- Call `requestAttackMode(agreement)` (from BCScript)
- Log state transition info (ATTACK_REQUESTED = 2, UNDER_ATTACK = 3)
- Include a `cast call` command comment for checking status
