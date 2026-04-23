# Branching Tree Technique

Credit for this to [Paul R Berg](https://x.com/PaulRBerg/status/1682346315806539776)

When creating tests, use the branching tree technique:

- Target a function
- Create a `.tree` file
- Consider all possible execution paths 
- Consider what contract state leads to what path
- Consider what function params lead to what paths
- Define "given state is x" nodes
- Define "when parameter is x" node 
- Define final "it should" tests

## Example Tree

```
├── when the id references a null stream
│   └── it should revert
└── when the id does not reference a null stream
    ├── given assets have been fully withdrawn
    │   └── it should return DEPLETED
    └── given assets have not been fully withdrawn
        ├── given the stream has been canceled
        │   └── it should return CANCELED
        └── given the stream has not been canceled
            ├── given the start time is in the future
            │   └── it should return PENDING
            └── given the start time is not in the future
                ├── given the refundable amount is zero
                │   └── it should return SETTLED
                └── given the refundable amount is not zero
                    └── it should return STREAMING
```

## Example Tests

```solidity
function test_RevertWhen_Null() external {
    uint256 nullStreamId = 1729;
    vm.expectRevert(abi.encodeWithSelector(Errors.SablierV2Lockup_Null.selector, nullStreamId));
    lockup.statusOf(nullStreamId);
}

modifier whenNotNull() {
    defaultStreamId = createDefaultStream();
    _;
}

function test_StatusOf()
    external
    whenNotNull
    givenAssetsNotFullyWithdrawn
    givenStreamNotCanceled
    givenStartTimeNotInFuture
    givenRefundableAmountNotZero
{
    LockupLinear.Status actualStatus = lockup.statusOf(defaultStreamId);
    LockupLinear.Status expectedStatus = LockupLinear.Status.STREAMING;
    assertEq(actualStatus, expectedStatus);
}
```
