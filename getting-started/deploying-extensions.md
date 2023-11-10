# Deploying Extensions

## Overview

Extensions add additional functionality to a vault. Extensions can range from allowing every possible transaction, all the way to explicitly locking down the functionality of a vault to a single smart contract function call.

This allows a large spectrum of strategies to be built, however the degree of trust in a vault operator grows substantially as they move away from clearly defined and relatively secure permissions.&#x20;

An example of a tightly scoped extension that requires less trust in the operator is functionality that enables trading on Uniswap with a pre-approved pair of tokens.&#x20;

As you can see the extension checks tokens in/out along with verifies that the recipient of tokens is the vault. No other Uniswap actions are allowed.&#x20;

```solidity
function swapExactInputSingle(
    ISwapRouter.ExactInputSingleParams memory params
) external returns (uint256) {
    require(
        params.tokenIn == token0 || params.tokenIn == token1,
        "token not allowed"
    );
    require(
        params.tokenOut == token0 || params.tokenOut == token1,
        "token not allowed"
    );

    require(params.recipient == address(this), "unauthorized recipient");

    // optionally handle validation
    return swapRouter.exactInputSingle(params);
}
```

## Deployment

In this example we are using Arbitrum to keep costs low, while still having a real testing environment with actual liquidity and trading activity.\
\
The script we are going to run is called **CreateUniswapSinglePair**. If you desire to deploy on a different network, it's important to replace the hardcoded addresses with correct addresses for the respective network:

```solidity
contract CreateUniswapSinglePair is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        address swapRouter = address(
            0xE592427A0AEce92De3Edee1F18E0157C05861564
        );
        address nonfungiblePositionManager = address(
            0xC36442b4a4522E871399CD717aBDD847Ab11FE88
        );
        // WETH
        address token0 = address(0x82aF49447D8a07e3bd95BD0d56f35241523fBab1);
        // USDC
        address token1 = address(0xaf88d065e77c8cC2239327C5EDb3A432268e5831);

        UniswapSinglePair parameter = new UniswapSinglePair(
            swapRouter,
            nonfungiblePositionManager,
            token0,
            token1
        );

        vm.stopBroadcast();
    }
}
```

The **UniswapSinglePair** extensions contract enables all the functionality of Uniswap v3 (swapping, managing liquidity, etc.) but limits it to a specific pair of tokens (WETH and USDC in our case).&#x20;

```bash
source .env && forge script CreateUniswapSinglePair --rpc-url $RPC_URL --broadcast --verify
```

At this point make sure you have both the address for your vault, and the address for this uniswap single pair extension contract.
