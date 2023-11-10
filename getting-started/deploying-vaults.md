# Deploying Vaults

## Overview

The core behind parameter is an asset vault. Every vault has a base asset that is used for deposits, withdrawals, and relative performance metrics. \
\
Besides the logic for interacting with LPs (deposit, withdrawal requests, repossession, etc.), the core of the vault operates similar to a [diamond proxy contract](https://eips.ethereum.org/EIPS/eip-2535). \
\
The vault delegate calls specific permissions contracts that enable functionality. An example of this would be a contract that extends the functionality of the vault so that it can interact with uniswap, seaport, and friend.tech contracts.

<figure><img src="../.gitbook/assets/Screen Shot 2023-11-10 at 10.57.45 AM.png" alt=""><figcaption><p>The vault contract delegate calls contracts that extend the functionality of the vault.</p></figcaption></figure>



## Deployment



Start by cloning the vaults repo:\
[https://github.com/parameterlabs/vaults](https://github.com/parameterlabs/vaults)

```
git clone https://github.com/parameterlabs/vaults.git
```

At the base of the repo, you will need to add a .env file with the following structure:

```bash
# .env
RPC_URL=YOUR_RPC_URL
PRIVATE_KEY=YOUR_PRIVATE_KEY
ETHERSCAN_API_KEY=YOUR_ETHERSCAN_KEY
```



Inside of the repo is a folder called scripts. The first script we will use is the **CreateVault** script.

```solidity
contract CreateVault is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        Vault vault = new Vault(
            address(YOUR_ADDRESS_HERE)
        );

        vm.stopBroadcast();
    }
}
```

Be sure to replace the owner address in the script to be the same as your public key. \
\
Then run the following:

```
source .env && forge script CreateVault --rpc-url $RPC_URL --broadcast --verify
```
