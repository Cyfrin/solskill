# Solidity Development Standards

Instructions for writing production grade solidity code and working with BattleChain.

# Installation/Quickstart

## Solidity

```bash
npx skills add cyfrin/solskill --skill solidity
```

## All skills

```bash
npx skills add cyfrin/solskill 
```

## Battle-tested deployments (battlechain)

```bash
npx skills add cyfrin/solskill --skill battlechain
npx skills add cyfrin/solskill --skill battlechain-tutorial
```

## Marketplace

```bash
# Open a claude code terminal, and in the claude prompt run:
/plugin marketplace add Cyfrin/solskill

# Install individual skills:
/plugin install solidity@solskill
/plugin install battlechain@solskill
/plugin install battlechain-tutorial@solskill

# Press Ctrl+C then run claude again. Or type /exit to quit first.
/exit
claude --continue
```

# Skills

## Solidity

Production-grade Solidity development standards covering code quality, testing patterns, security practices, and Foundry workflows.

```
Create me an AMM with 2 tokens and a 0.3% fee
Build a token vault contract for a specific token
```

## BattleChain

Reference for [BattleChain](https://docs.battlechain.com/) — the pre-mainnet L2 for battle-testing smart contracts with real funds. Covers deploying contracts, creating Safe Harbor agreements, whitehat attack workflows, contract lifecycle management, and on-chain APIs.

```
How do I request attack mode for my contract on BattleChain?
Find contracts under attack on BattleChain and check their bounty terms
```

## BattleChain Tutorial

Interactive deployment wizard that walks you through preparing your Foundry project for BattleChain. Scans your existing contracts and scripts, asks 14 guided questions with clickable options, then generates all the deployment scripts and Safe Harbor agreements for you.

```
Deploy my contracts to BattleChain
Write smart contract scripts for BattleChain
```

# Learn more

- [Cyfrin forge template](https://github.com/Cyfrin/forge-template/) for setting up a basic Foundry project
- [BattleChain docs](https://docs.battlechain.com/) for the full BattleChain documentation
- [BattleChain starter](https://github.com/Cyfrin/battlechain-starter) for a starter project to deploy to BattleChain

# Authors

[Patrick Collins](https://x.com/PatrickAlphaC) and the [Cyfrin](https://www.cyfrin.io/) team.