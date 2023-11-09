# Setup Polygon Supernets v1.3 on Ubuntu 22
This setup is for manual deployments for cases where an ERC20 Rootchain Token is mapped to the native Supernet Token.

Last Updated: Nov 9, 2023
Release Version: 1.3.2
Subrelease Version: Commit SHA ab9b887

## Prequisites:
- Golang 1.20

## 1. Clone and Build Supernets
```
git clone https://github.com/0xPolygon/polygon-edge
cd polygon-edge
```

In case you'd like use a specific commit of Supernets, you can switch it by
```
git checkout -b supernets <commit_hash>
```

Next we compile the `polygon-edge` binary
```
go build -o polygon-edge .
```

## 2. Initialize secrets (Insecure)
```
./polygon-edge polybft-secrets --data-dir test-chain- --num 4 --insecure
```

## 3. Initialize Shell Variables
This will allow you to refer the same set of variables for the next commands. Note that closing the terminal resets the shell variables. Ideally the Deployer Address should be funded with sufficient native tokens on the rootchain to cover for gas costs. 

The Rootchain Stake Token is an ERC20 that's deployed on the rootchain. This token is used by validators for staking.

The Rootchain Parent Token is yet another ERC20 on the rootchain that acts as the parent token for the native token on the Supernet.

The Reward Token Code is the byteCode for an ERC20 contract which will serve as Validator rewards on the Supernet if native Supernet token minting is set to false.

The Proxy Contract Admin Address should be different from the Deployer's Address
```
DEPLOYER_ADDRESS=<deployer_wallet_address>
DEPLOYER_KEY=<hex_encoded_deployer_key>
ROOTCHAIN_RPC=<rootchain_rpc_here>
ROOTCHAIN_STAKE_TOKEN=<stake_token_address_here>
ROOTCHAIN_PARENT_TOKEN=<parent_token_address_here>
PROXY_CONTRACT_ADMIN=<proxy_contract_admin_address_here>
REWARD_TOKEN_CODE=<ERC20_contract_bytecode_here>
```

## 4. Initialize genesis file with allowlisting / blocklisting, premining, and native token config
To allowlist specific addresses to make transactions to your Supernet, you can use the `--transactions-allow-list-admin` flag to specify the allowlist admins and `--transactions-allow-list-enabled` flag to list the allowed addresses.

To blocklist specific addresses from making transactions to your Supernet, you can use the `--transactions-block-list-admin` flag to specify the blocklist admins and `--transactions-block-list-enabled` flag to list the blocked addresses.

*Note*: It is recommended to either use an allowlist or blocklist at a time. Using both or neither will cause unforeseen issues. The same holds true for the `--contract-deployer-allow-list-admin`,`--contract-deployer-allow-list-enabled`,`--contract-deployer-block-list-admin`,`--contract-deployer-block-list-enabled`,`--bridge-allow-list-admin`,`--bridge-allow-list-enabled`,`--bridge-block-list-admin`,`--bridge-block-list-enabled` flags.

The `--native-token-config` file sets the attributes of the native token of the Supernet. In case the minting flag inside the native token config is set to `true` inside the config, the minter's address needs to be supplied at the end.
```
./polygon-edge genesis --block-gas-limit 0 --block-time 6s --chain-id 7567 --consensus polybft --epoch-size 10 --name my_supernet --native-token-config "SuperETH:SETH:18:false" --reward-wallet 0x1:1 --reward-token-code $REWARD_TOKEN_CODE --premine 0x0:1 --validators-path ./ --validators-prefix test-chain- --bridge-block-list-admin $DEPLOYER_ADDRESS --bridge-block-list-enabled 0x42 --contract-deployer-block-list-admin $DEPLOYER_ADDRESS --contract-deployer-block-list-enabled 0x42 --transactions-block-list-admin $DEPLOYER_ADDRESS --transactions-block-list-enabled 0x42 --proxy-contracts-admin $PROXY_CONTRACT_ADMIN
```

```
VALIDATOR_1=$(jq -r '.params.engine.polybft.initialValidatorSet[0].address' "genesis.json")
VALIDATOR_2=$(jq -r '.params.engine.polybft.initialValidatorSet[1].address' "genesis.json")
VALIDATOR_3=$(jq -r '.params.engine.polybft.initialValidatorSet[2].address' "genesis.json")
VALIDATOR_4=$(jq -r '.params.engine.polybft.initialValidatorSet[3].address' "genesis.json")
```

**Note**: In case you'd like to prefund an account on your Supernet, you can use this shell command to assign some balance to any address you'd like. In this example we use `$DEPLOYER_ADDRESS`
```
jq --arg key "$DEPLOYER_ADDRESS" '.genesis.alloc += { ($key): { "balance": "0x56bc75e2d63100000"  } }' genesis.json > temp.json && mv temp.json genesis.json
```

Similarly, we need to fund the validator addresses to ensure they have enough funds for bridging

```
jq --arg key "$VALIDATOR_1" '.genesis.alloc += { ($key): { "balance": "0x56bc75e2d63100000"  } }' genesis.json > temp.json && mv temp.json genesis.json

jq --arg key "$VALIDATOR_2" '.genesis.alloc += { ($key): { "balance": "0x56bc75e2d63100000"  } }' genesis.json > temp.json && mv temp.json genesis.json

jq --arg key "$VALIDATOR_3" '.genesis.alloc += { ($key): { "balance": "0x56bc75e2d63100000"  } }' genesis.json > temp.json && mv temp.json genesis.json

jq --arg key "$VALIDATOR_4" '.genesis.alloc += { ($key): { "balance": "0x56bc75e2d63100000"  } }' genesis.json > temp.json && mv temp.json genesis.json
```

## 5. Deploy StakeManager contract to Rootchain
Next we deploy the StakeManager on the Rootchain

**Note**: You have the option of running your own rootchain by opening a new terminal session and using the `./polygon-edge rootchain server` command. This will start a local geth node and expose it's RPC on `http://127.0.0.1:8545`.

**Note**: You need to deploy the `Rootchain Stake Token` with your private key before proceeding with this step. The `Rootchain Stake Token` is a simple ERC20 token contract which will be used by rootchain validators for staking.
```
./polygon-edge polybft stake-manager-deploy --private-key $DEPLOYER_KEY --genesis ./genesis.json --jsonrpc $ROOTCHAIN_RPC --stake-token $ROOTCHAIN_STAKE_TOKEN --proxy-contracts-admin $PROXY_CONTRACT_ADMIN
``` 
We should also assign the `STAKE_MANAGER` shell variable from the `genesis.json` file.
```
STAKE_MANAGER=$(jq -r '.params.engine.polybft.bridge.stakeManagerAddr' "genesis.json")
```

## 6. Deploy and initialize Rootchain contracts
This command deploys rootchain smart contracts and initializes them. It also updates `genesis.json` with rootchain contract addresses and rootchain default sender address.
```
./polygon-edge rootchain deploy --deployer-key $DEPLOYER_KEY --stake-manager $STAKE_MANAGER --stake-token $ROOTCHAIN_STAKE_TOKEN --erc20-token $ROOTCHAIN_PARENT_TOKEN --genesis ./genesis.json --json-rpc $ROOTCHAIN_RPC --proxy-contracts-admin $PROXY_CONTRACT_ADMIN
```

## 7. Fund validators on Rootchain
in order for validators to be able to send transactions to Ethereum, they need to be funded in order to be able to cover gas cost.

**Note**: 0.1ETH of native rootchain tokens will be transferred from the deployer's address to each of the validators. In case you wish to edit the funding amounts, refer the `--amounts` flag 
```
./polygon-edge rootchain fund --addresses $VALIDATOR_1,$VALIDATOR_2,$VALIDATOR_3,$VALIDATOR_4 --amounts 100000000000000000,100000000000000000,100000000000000000,100000000000000000 --json-rpc $ROOTCHAIN_RPC --private-key $DEPLOYER_KEY
```

## 8. Whitelist validators on Rootchain
In order for validators to be able to be registered on the SupernetManagaqer contract on rootchain. 

**Note**: only deployer of SupernetManager contract (the one who run the deploy command) can whitelist validators on rootchain. They can use either their hex encoded private key, or `data-dir` flag if they have screts initialised. 

The `customSupernetManagerAddr` can be found in the `genesis.json` file.
```
SUPERNET_MANAGER=$(jq -r '.params.engine.polybft.bridge.customSupernetManagerAddr' "genesis.json")
```
```
./polygon-edge polybft whitelist-validators --private-key $DEPLOYER_KEY --addresses $VALIDATOR_1,$VALIDATOR_2,$VALIDATOR_3,$VALIDATOR_4 --supernet-manager $SUPERNET_MANAGER --jsonrpc $ROOTCHAIN_RPC
```

## 9. Register validators on the Rootchain
Each validator registers itself on the rootchain.
```
./polygon-edge polybft register-validator --data-dir ./test-chain-1 --supernet-manager $SUPERNET_MANAGER --jsonrpc $ROOTCHAIN_RPC

./polygon-edge polybft register-validator --data-dir ./test-chain-2 --supernet-manager $SUPERNET_MANAGER --jsonrpc $ROOTCHAIN_RPC

./polygon-edge polybft register-validator --data-dir ./test-chain-3 --supernet-manager $SUPERNET_MANAGER --jsonrpc $ROOTCHAIN_RPC

./polygon-edge polybft register-validator --data-dir ./test-chain-4 --supernet-manager $SUPERNET_MANAGER --jsonrpc $ROOTCHAIN_RPC
```

## 10. Initial Staking on the Rootchain.
First use your private key to transfer some `Rootchain Staking Tokens` to your validators on the rootchain. After ensuring that the validators are funded with Rootchain Staking Tokens, we can begin staking on the rootchain.
```
SUPERNET_ID=$(jq -r '.params.engine.polybft.supernetID' "genesis.json")
```
**Note**: 0.1ETH of Rootchain Staking Tokens will be staked from the validator's address. In case you wish to edit the amount, refer the `--amount` flag
```
./polygon-edge polybft stake --data-dir ./test-chain-1 --supernet-id $SUPERNET_ID --amount 100000000000000000 --stake-manager $STAKE_MANAGER --stake-token $ROOTCHAIN_STAKE_TOKEN --jsonrpc $ROOTCHAIN_RPC

./polygon-edge polybft stake --data-dir ./test-chain-2 --supernet-id $SUPERNET_ID --amount 100000000000000000 --stake-manager $STAKE_MANAGER --stake-token $ROOTCHAIN_STAKE_TOKEN --jsonrpc $ROOTCHAIN_RPC

./polygon-edge polybft stake --data-dir ./test-chain-3 --supernet-id $SUPERNET_ID --amount 100000000000000000 --stake-manager $STAKE_MANAGER --stake-token $ROOTCHAIN_STAKE_TOKEN --jsonrpc $ROOTCHAIN_RPC

./polygon-edge polybft stake --data-dir ./test-chain-4 --supernet-id $SUPERNET_ID --amount 100000000000000000 --stake-manager $STAKE_MANAGER --stake-token $ROOTCHAIN_STAKE_TOKEN --jsonrpc $ROOTCHAIN_RPC
```

## 11. Finalize Genesis Validator set on Rootchain (Supernet Manager) contract.
This is done after all validators from genesis do initial staking on rootchain, and it's a final step that is required before starting the child chain. This needs to be done by the deployer of SupernetManager contract (the user that run the deploy command).

If enable-staking flag is provided, validators will be able to continue staking on rootchain. If not, genesis validators will not be able update its stake or unstake, nor will newly registered validators after genesis will be able to stake tokens on the rootchain. Enabling of staking can be done through this command, or later after the child chain starts.
```
./polygon-edge polybft supernet --private-key $DEPLOYER_KEY --genesis ./genesis.json --supernet-manager $SUPERNET_MANAGER --stake-manager $STAKE_MANAGER --finalize-genesis-set --enable-staking --jsonrpc $ROOTCHAIN_RPC
```

## 12. Run (child chain) cluster in relayer mode
In this example our chain chain cluster consists of 4 Edge clients. You can use the `--relayer` flag to run child chain nodes in `relayer` mode. It allows automatic execution of deposit events on behalf of users. Open 4 terminal windows to run each node:
```
./polygon-edge server --data-dir ./test-chain-1 --chain genesis.json --grpc-address :5001 --libp2p :30301 --jsonrpc :9545 --log-level DEBUG --relayer
```
```
./polygon-edge server --data-dir ./test-chain-2 --chain genesis.json --grpc-address :5002 --libp2p :30302 --jsonrpc :10002 --log-level DEBUG --relayer
```
```
./polygon-edge server --data-dir ./test-chain-3 --chain genesis.json --grpc-address :5003 --libp2p :30303 --jsonrpc :10003 --log-level DEBUG --relayer
```
```
./polygon-edge server --data-dir ./test-chain-4 --chain genesis.json --grpc-address :5004 --libp2p :30304 --jsonrpc :10004 --log-level DEBUG --relayer
```