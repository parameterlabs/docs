# Making Swaps

The final step is to verify that our extension works as expected. Included in the repo is a swap script that calls a Uniswap swap with our pre-approved tokens.

## Funding

Start by sending some WETH tokens to your vault. For speeds sake, its easier to wrap to WETH using your wallet and send the weth tokens directly to the vault.&#x20;

## Approvals

Because the vault is using ERC20's to trade, we need to approve both tokens for trade. The following script accomplishes this. Be sure to change the vault contract out for your own.

```solidity
contract Approve is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        UniswapSinglePair vault = UniswapSinglePair(
            payable(YOUR_VAULT_CONTRACT)
        );

        address weth = address(0x82aF49447D8a07e3bd95BD0d56f35241523fBab1);
        address usdc = address(0xaf88d065e77c8cC2239327C5EDb3A432268e5831);

        vault.approveSwapRouter(weth, type(uint).max);
        vault.approveSwapRouter(usdc, type(uint).max);

        vm.stopBroadcast();
    }
}
```

```
source .env && forge script Approve --rpc-url $RPC_URL --broadcast   
```

## Swap

**Note**: you may have to adjust the Uniswap **ExactInputSingleParams** according to market conditions.  More information can be found here: \
\
[https://docs.uniswap.org/contracts/v3/guides/swaps/single-swaps](https://docs.uniswap.org/contracts/v3/guides/swaps/single-swaps)



The swap contract executes a swap by routing through our vault contract. Because of the delegate calls, even though the uniswap extension is a seperate contract, it appears to extend the functionality of the vault, meaning we can use funds contained inside the vault.&#x20;

```solidity
contract Swap is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        address payable vaultAddress = payable(
            0xfa0F0978b43b54c5AeD816D2a324304829934e1b
        );

        UniswapVaultInterface vault = UniswapVaultInterface(vaultAddress);

        address weth = address(0x82aF49447D8a07e3bd95BD0d56f35241523fBab1);
        address usdc = address(0xaf88d065e77c8cC2239327C5EDb3A432268e5831);
        uint amountIn = 451891333259520;

        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter
            .ExactInputSingleParams({
                tokenIn: weth,
                tokenOut: usdc,
                fee: 3000,
                recipient: vaultAddress,
                deadline: block.timestamp + 100,
                amountIn: amountIn,
                amountOutMinimum: 847062,
                sqrtPriceLimitX96: 0
            });

        vault.swapExactInputSingle(params);
        vm.stopBroadcast();
    }
}
```

**Be sure to replace the vault address with your own, and adjust the amounts and swap params according to your specific situation.**

Finally run this script:

```
source .env && forge script Swap --rpc-url $RPC_URL --broadcast
```

As an exercise, try changing the **tokenOut** parameter to another token. The transaction should get blocked by our extension, thus enforcing the rule of trading with a specific pair of tokens.
