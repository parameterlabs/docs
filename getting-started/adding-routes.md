# Adding Routes

## Overview

While we prototype vault contracts, we allow the vault owner to arbitrarily add and remove extension routes. In the future, there will be a lock so that extensions can only be added or removed prior to receiving any funds. By explicitly locking extensions, the investors in a vault know precisely what functionality is allowed.&#x20;

But since we are testing, we don't need to worry about that!



The most important function in vault contract is the fallback function:

```solidity
fallback() external payable {
    bytes4 functionSelector = bytes4(msg.data[:4]);
    address target = routes[functionSelector];
    if (target != address(0) && msg.sender == owner()) {
        (bool success, ) = target.delegatecall(msg.data);
        require(success, "Delegatecall failed");
    }
}
```

If you read carefully, you'll notice that the fallback effectively looks for a function selector and routes that function call to the desired extension, otherwise doing nothing. \
\
To add functionality to our vault, we need to set the routes mapping. The add routes script handles the nitty gritty of getting each function selector and setting the contract address to route to.&#x20;

**Be sure to change out the vault contract, and permissions contract according to your own deployments.**

```solidity
contract AddRoutes is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        Vault vault = Vault(
            payable(0xfa0F0978b43b54c5AeD816D2a324304829934e1b)
        );

        // CHANGE ME!!!!
        address permissionsContract = address(
            0xd6d650Bef12989926b2d5ECb62DD98f509e8da1d
        );

        bytes4[] memory FUNCTION_SELECTORS = new bytes4[](9);
        FUNCTION_SELECTORS[0] = UniswapSinglePair
            .approvePositionManager
            .selector;
        FUNCTION_SELECTORS[1] = UniswapSinglePair.approveSwapRouter.selector;
        FUNCTION_SELECTORS[2] = UniswapSinglePair.swapExactInputSingle.selector;
        FUNCTION_SELECTORS[3] = UniswapSinglePair
            .swapExactOutputSingle
            .selector;
        FUNCTION_SELECTORS[4] = UniswapSinglePair
            .mintLiquidityPosition
            .selector;
        FUNCTION_SELECTORS[5] = UniswapSinglePair.collectLiquidityFees.selector;
        FUNCTION_SELECTORS[6] = UniswapSinglePair.increaseLiquidity.selector;
        FUNCTION_SELECTORS[7] = UniswapSinglePair.decreaseLiquidity.selector;
        FUNCTION_SELECTORS[8] = UniswapSinglePair.onERC721Received.selector;

        address[] memory TARGET_CONTRACTS = new address[](9);
        TARGET_CONTRACTS[0] = permissionsContract;
        TARGET_CONTRACTS[1] = permissionsContract;
        TARGET_CONTRACTS[2] = permissionsContract;
        TARGET_CONTRACTS[3] = permissionsContract;
        TARGET_CONTRACTS[4] = permissionsContract;
        TARGET_CONTRACTS[5] = permissionsContract;
        TARGET_CONTRACTS[6] = permissionsContract;
        TARGET_CONTRACTS[7] = permissionsContract;
        TARGET_CONTRACTS[8] = permissionsContract;

        vault.addRoutes(FUNCTION_SELECTORS, TARGET_CONTRACTS);

        vm.stopBroadcast();
    }
}

```

Then go ahead and run the script with this:

```bash
source .env && forge script AddRoutes --rpc-url $RPC_URL --broadcast
```
