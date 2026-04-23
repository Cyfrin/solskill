# Governance

Use [safe-utils](https://github.com/Recon-Fuzz/safe-utils) or equivalent tooling for governance proposals. This makes multisig interactions testable, auditable, and reproducible through Foundry scripts rather than manual UI clicks. If you must use a UI, it's preferred to keep your transactions private, using a UI like [localsafe.eth](https://github.com/Cyfrin/localsafe.eth).

Write fork tests that verify expected protocol state after governance proposals execute. Fork testing against mainnet state catches misconfigurations that unit tests miss — for example, the [Moonwell price feed misconfiguration](https://rekt.news/moonwell-rekt) would have been caught by a fork test asserting correct oracle state post-proposal.

```solidity
// good - fork test verifying governance proposal outcome
function testGovernanceProposal_UpdatesPriceFeed() public {
    vm.createSelectFork(vm.envString("MAINNET_RPC_URL"));

    // Execute the governance proposal
    _executeProposal(proposalId);

    // Verify expected state after proposal
    address newFeed = oracle.priceFeed(market);
    assertEq(newFeed, EXPECTED_CHAINLINK_FEED);

    // Verify the feed returns sane values
    (, int256 price,,,) = AggregatorV3Interface(newFeed).latestRoundData();
    assertGt(price, 0);
}
```
