# BattleChain Deployment — Question Flow

This file contains the full question flow for gathering deployment parameters. Ask questions **one at a time** using the `AskUserQuestion` tool. Wait for each answer before proceeding. If a user's answer naturally covers upcoming questions, acknowledge and skip ahead.

## AskUserQuestion Usage Pattern

All questions use the `AskUserQuestion` tool with these rules (do NOT repeat these instructions per question):

- The tool automatically adds an "Other" option for free-text input.
- Support 2-4 options per question. Pick the most common/useful choices.
- For free-form inputs (addresses, names, numbers), provide sensible defaults so the user can pick or type their own.
- **Custom answer confirmation:** When the user selects "Other", confirm what you understood with a yes/no `AskUserQuestion` before moving on. Re-ask if they say no.
- Use `multiSelect: true` where noted.

---

## Question 0 — Target Chain

> "Where are you deploying these contracts?"

| Option | Description |
|--------|-------------|
| BattleChain | Deploy to BattleChain testnet (chain 627) -- full lifecycle with whitehats |
| Another L2 | Deploy to a different EVM L2 -- I'll help you set up CreateX and Safe Harbor too |
| Both | Deploy to BattleChain AND another L2 |

- If **BattleChain only**: skip to Question 1.
- If **Another L2** or **Both**: ask 0a and 0b first.

## Question 0a — CreateX Deployment (non-BattleChain only)

> "Do you want to use CreateX for deterministic contract addresses on your L2? This gives you the same addresses across all chains."

| Option | Description |
|--------|-------------|
| Yes (Recommended) | Use CreateX (0xba5Ed...) -- deployed on 190+ EVM chains for deterministic CREATE2/CREATE3 |
| No | Use standard deployment -- addresses will differ per chain |

## Question 0b — Safe Harbor Agreement (non-BattleChain only)

> "Do you want to create a Safe Harbor agreement for your non-BattleChain deployment too? This protects whitehats who responsibly disclose vulnerabilities."

| Option | Description |
|--------|-------------|
| Yes (Recommended) | Create a Safe Harbor agreement on your L2 using the SEAL Safe Harbor V3 URI |
| Not yet | Skip for now -- you can add one later |

If "Yes": bounty/agreement questions (3-13) apply to both chains. Collect answers once and generate agreements for each target chain. BattleChain uses `BATTLECHAIN_SAFE_HARBOR_URI`; non-BattleChain uses `SAFE_HARBOR_V3_URI`.

---

## Question 1 — Contract Inventory

**Pre-step:** Scan `src/**/*.sol` with `Glob`. Present discovered contracts with `multiSelect: true`. List up to 4 as options (group/summarize if more).

Example options:
- Label: "Token.sol", Description: "src/Token.sol -- ERC20 token contract"
- Label: "Vault.sol", Description: "src/Vault.sol -- Vault contract"

If existing scripts reveal deployment order or constructor arguments, mention it: e.g. "I see from your existing scripts that Token is deployed first, then passed to Vault's constructor -- I'll replicate that flow."

After selection, read each chosen file for constructor arguments and initialization parameters. If existing scripts already show what constructor args and init calls are used, confirm with the user rather than asking from scratch. Ask follow-ups for any missing constructor arguments.

## Question 2 — External Contract Dependencies

Only ask if the pre-scan identified external contracts the protocol interacts with (e.g. Uniswap, Chainlink, WETH, governance). Skip entirely if none found.

For each external dependency, use `AskUserQuestion`:
> "Your contracts interact with [ExternalContract] (mainnet: [address if known]). What's the BattleChain address for this?"

| Option | Description |
|--------|-------------|
| I have the address | Provide the BattleChain address (via Other) |
| Deploy a mock | Deploy a mock/stub version of this contract on BattleChain |
| Skip -- not needed | This dependency isn't needed for the BattleChain deployment |

Repeat for each dependency. May batch up to 4 into a single call or ask one at a time.

## Question 3 — Contracts in Scope (for Safe Harbor)

Use `multiSelect: true`, listing contracts selected in Question 1. Ask which should be in scope for whitehat attacks.

Then analyze selected in-scope contracts for child contract deployments (look for `new`, `create`, `create2`, `deploy` calls, or factory patterns). If child contracts are found:

> "I found that [ContractName] deploys the following child contracts: [ChildA], [ChildB]. Which of these should be in scope for whitehat attacks?"

| Option | Description |
|--------|-------------|
| All | All child contracts ([list them]) automatically in scope |
| None | Only the exact parent contracts listed -- no children |
| Exact | Only specific child addresses (you'll provide them later) |

Maps to `ScopeAccount.childContractScope`: `All`, `None`, or `Exact`.

If no child contracts detected, state: "None of the selected contracts deploy child contracts, so child scope doesn't apply." and skip this sub-question.

## Question 4 — Asset Recovery Address

> "Where should recovered funds be sent if a whitehat drains a contract?"

| Option | Description |
|--------|-------------|
| Deployer address | Use your deployer wallet address |
| Multisig / Treasury | A separate multisig or treasury address (specify via Other) |

## Question 5 — Bounty Percentage

> "What percentage of drained funds should the whitehat keep as a bounty?"

| Option | Description |
|--------|-------------|
| 5% | Conservative -- lower incentive |
| 10% | Standard bounty percentage (Recommended) |
| 15% | Generous -- strong incentive |
| 20% | Very generous -- maximum incentive |

## Question 6 — Bounty Cap (USD)

> "Maximum USD bounty cap per exploit?"

| Option | Description |
|--------|-------------|
| $500K | Five hundred thousand dollar cap |
| $1M | One million dollar cap |
| $5M | Five million dollar cap |
| No cap | No limit on per-exploit bounty |

## Question 7 — Aggregate Bounty Cap (USD)

> "Aggregate cap across ALL exploits during the attack window?"

| Option | Description |
|--------|-------------|
| $1M | One million dollar total cap |
| $5M | Five million dollar total cap |
| $10M | Ten million dollar total cap |
| No cap | No aggregate limit -- potentially unlimited total payout |

## Question 8 — Funds Retainable

> "Can the whitehat keep their bounty on the spot?"

| Option | Description |
|--------|-------------|
| Yes | They keep their percentage immediately |
| No | All funds returned first, bounty paid separately |

## Question 9 — Identity Requirements

> "Do whitehats need to identify themselves to claim a bounty?"

| Option | Description |
|--------|-------------|
| Anonymous | No identity required |
| Pseudonymous | On-chain identity only |
| Named | Real-world KYC required |

## Question 10 — Diligence Requirements

> "Any specific requirements whitehats must follow before attacking?"

| Option | Description |
|--------|-------------|
| Check mainnet first | Must verify the vulnerability doesn't exist on mainnet |
| None | No special requirements |

## Question 11 — Protocol Name & Contact

Ask as two separate sub-questions.

**11a. Protocol name:**
> "What's your protocol's name?"

| Option | Description |
|--------|-------------|
| Use repo name | Derive protocol name from the repository name |
| Custom | Specify a custom protocol name (via Other) |

**11b. Security contact email:**
> "What's your security contact email?"

| Option | Description |
|--------|-------------|
| Use @custom-contact.xyz | Placeholder -- type your real contact via Other |

The user should always type their own contact information.

## Question 12 — Agreement URI

> "Do you have a legal Safe Harbor document URI?"

| Option | Description |
|--------|-------------|
| Skip for now | No agreement URI -- can be added later |
| Yes, I have one | Provide an IPFS hash or URL (via Other) |

## Question 13 — Commitment Window

> "How many days do you commit to not worsening bounty terms? (minimum 7)"

| Option | Description |
|--------|-------------|
| 7 days | Minimum commitment period |
| 14 days | Two-week commitment |
| 30 days | One-month commitment (Recommended) |
| 90 days | Three-month commitment |

## Question 14 — Seed Amount

> "How much of your token (in whole units) will you seed as starting liquidity?"

| Option | Description |
|--------|-------------|
| 1,000 | One thousand tokens |
| 10,000 | Ten thousand tokens |
| 100,000 | One hundred thousand tokens |
| 1,000,000 | One million tokens |
