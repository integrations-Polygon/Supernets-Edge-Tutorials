# Setup Polygon Supernets v1.0.0 on Ubuntu 22
Last Updated: Jun 20, 2023
Release Version: 1.0.0
Subrelease Version: Commit de084a9

## Prequisites:
- Golang 1.19
- Docker

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

## 3. Initialize genesis file with allowlisting / blocklisting, premining, and native token config
To allowlist specific addresses to make transactions to your Supernet, you can use the `--transactions-allow-list-admin` flag to specify the allowlist admins and `--transactions-allow-list-enabled` flag to list the allowed addresses.

To blocklist specific addresses from making transactions to your Supernet, you can use the `--transactions-block-list-admin` flag to specify the blocklist admins and `--transactions-block-list-enabled` flag to list the blocked addresses.


*Note*: It is recommended to either use an allowlist or blocklist at a time. Using both or neither will cause unforeseen issues. The same holds true for the `--contract-deployer-allow-list-admin`,`--contract-deployer-allow-list-enabled`,`--contract-deployer-block-list-admin`,`--contract-deployer-block-list-enabled`,`--bridge-allow-list-admin`,`--bridge-allow-list-enabled`,`--bridge-block-list-admin`,`--bridge-block-list-enabled` flags.

The `--native-token-config` file sets the attributes of the native token of the Supernet. In case the minting flag inside the native token config is set to `true` inside the config, the minter's address needs to be supplied at the end.
```
./polygon-edge genesis /
--block-gas-limit 10000000 --block-time 6s /
--epoch-size 10 /
--validators-path ./ /
--validators-prefix test-chain- /
--consensus polybft /
--reward-wallet <your_wallet_address_here>:1000000 /

[--transactions-allow-list-admin <transactions_allow_list_admin_addresses_here> --transactions-allow-list-enabled <transactions_allowed_addresses_here>] /
[--transactions-block-list-admin <transactions_block_list_admin_addresses_here> --transactions-block-list-enabled <transactions_blocked_addresses_here>] /

[--contract-deployer-allow-list-admin <contract_deployer_allow_list_admin_here> --contract-deployer-allow-list-enabled <contract_deployer_allowed_addresses_here>] /
[--contract-deployer-block-list-admin <contract_deployer_block_list_admin_here> --contract-deployer-block-list-enabled <contract_deployer_blocked_addresses_here>] /

[--bridge-allow-list-admin <bridge_allow_list_admin_addresses_here> --bridge-allow-list-enabled <bridge_allowed_addresses_here>] /
[--bridge-block-list-admin <bridge_block_list_admin_addresses_here> --bridge-block-list-enabled <bridge_blocked_addresses_here>] /

--premine <your_wallet_address_here>:100000000000000000000 /
--native-token-config "SuperETH:SETH:18:true:<native_token_minter_address>"
```

## 4. Deploy StakeManager contract to Rootchain
Next we deploy the StakeManager on the Rootchain

**Note**: You have the option of running your own rootchain by opening a new terminal session and using the `./polygon-edge rootchain server` command. This will start a local geth node and expose it's RPC on `http://127.0.0.1:8545`.

**Note**: You need to deploy the `Rootchain Stake Token` with your private key before proceeding with this step. The `Rootchain Stake Token` is a simple ERC20 token contract which will be used by rootchain validators for staking.
```
./polygon-edge polybft stake-manager-deploy --private-key <your_hex_encoded_private_key_here> --genesis ./genesis.json --jsonrpc <rootchain_rpc_here> --stake-token <stake_token_address_here>
``` 
Note the `stakeManagerAddr` and `stakeTokenAddr` inside the `genesis.json` file

## 5. Deploy and initialize Rootchain contracts
This command deploys rootchain smart contracts and initializes them. It also updates `genesis.json` with rootchain contract addresses and rootchain default sender address.
```
./polygon-edge rootchain deploy --deployer-key <your_hex_encoded_private_key_here> --stake-manager <stakeManagerAddr_here> --stake-token <stakeTokenAddr> --genesis ./genesis.json --json-rpc <rootchain_rpc_here>
```

## 6. Fund validators on Rootchain
in order for validators to be able to send transactions to Ethereum, they need to be funded in order to be able to cover gas cost. This command is for testing purposes only.
```
 ./polygon-edge rootchain fund --addresses <validator_addresses_here> --amounts <funding_amounts_here> --json-rpc <rootchain_rpc_here> --private-key <your_hex_encoded_private_key_here>
```

## 7. Whitelist validators on Rootchain
In order for validators to be able to be registered on the SupernetManagaqer contract on rootchain. 

**Note**: only deployer of SupernetManager contract (the one who run the deploy command) can whitelist validators on rootchain. They can use either their hex encoded private key, or `data-dir` flag if they have screts initialised. 

The `customSupernetManagerAddr` can be found in the `genesis.json` file.
```
./polygon-edge polybft whitelist-validators --private-key <your_hex_encoded_private_key_here> --addresses <validator_address_here> --supernet-manager <customSupernetManagerAddr_here> --jsonrpc <rootchain_rpc_here>
```

## 8. Register validators on the Rootchain
Each validator registers itself on the rootchain.
```
./polygon-edge polybft register-validator --data-dir ./test-chain-1 --supernet-manager <customSupernetManagerAddr_here> --jsonrpc <rootchain_rpc_here>

./polygon-edge polybft register-validator --data-dir ./test-chain-2 --supernet-manager <customSupernetManagerAddr_here> --jsonrpc <rootchain_rpc_here>

./polygon-edge polybft register-validator --data-dir ./test-chain-3 --supernet-manager <customSupernetManagerAddr_here> --jsonrpc <rootchain_rpc_here>

./polygon-edge polybft register-validator --data-dir ./test-chain-4 --supernet-manager <customSupernetManagerAddr_here> --jsonrpc <rootchain_rpc_here>
```

## 9. Initial Staking on the Rootchain.
First use your private key to transfer some `Rootchain Staking Tokens` to your validators on the rootchain. After ensuring that the validators are funded with Rootchain Staking Tokens, we can begin staking on the rootchain:
```
./polygon-edge polybft stake --data-dir ./test-chain-1 --supernet-id <supernet_id_here> --amount <stake_amount_here> --stake-manager <stakeManagerAddr_here> --stake-token <stakeTokenAddr_here> --jsonrpc <rootchain_rpc_here>

./polygon-edge polybft stake --data-dir ./test-chain-2 --supernet-id <supernet_id_here> --amount <stake_amount_here> --stake-manager <stakeManagerAddr_here> --stake-token <stakeTokenAddr_here> --jsonrpc <rootchain_rpc_here>

./polygon-edge polybft stake --data-dir ./test-chain-3 --supernet-id <supernet_id_here> --amount <stake_amount_here> --stake-manager <stakeManagerAddr_here> --stake-token <stakeTokenAddr_here> --jsonrpc <rootchain_rpc_here>

./polygon-edge polybft stake --data-dir ./test-chain-4 --supernet-id <supernet_id_here> --amount <stake_amount_here> --stake-manager <stakeManagerAddr_here> --stake-token <stakeTokenAddr_here> --jsonrpc <rootchain_rpc_here>
```
The `supernet_id`, `stakeManagerAddr` and `stakeTokenAddr` values can be obtained from the `genesis.json` file.

## 10. Finalize Genesis Validator set on Rootchain (Supernet Manager) contract.
This is done after all validators from genesis do initial staking on rootchain, and it's a final step that is required before starting the child chain. 

This needs to be done by the deployer of SupernetManager contract (the user that run the deploy command). He can use either its hex encoded private key, or data-dir flag if he has secerets initialized. 

If enable-staking flag is provided, validators will be able to continue staking on rootchain. If not, genesis validators will not be able update its stake or unstake, nor will newly registered validators after genesis will be able to stake tokens on the rootchain. Enabling of staking can be done through this command, or later after the child chain starts.
```
./polygon-edge polybft supernet --private-key <your_hex_encoded_private_key_here> --genesis ./genesis.json --supernet-manager <customSupernetManagerAddr_here> --stake-manager <stakeManagerAddr_here> --finalize-genesis-set --enable-staking --jsonrpc <rootchain_rpc_here>
```

## 11. Run (child chain) cluster
In this example our chain chain cluster consists of 4 Edge clients. You can use the `--relayer` flag to run child chain nodes in `relayer` mode. It allows automatic execution of deposit events on behalf of users.
```
./polygon-edge server --data-dir ./test-chain-1 --chain genesis.json --grpc-address :5001 --libp2p :30301 --jsonrpc :9545 --seal --log-level DEBUG [--relayer]

./polygon-edge server --data-dir ./test-chain-2 --chain genesis.json --grpc-address :5002 --libp2p :30302 --jsonrpc :10002 --seal --log-level DEBUG [--relayer]

./polygon-edge server --data-dir ./test-chain-3 --chain genesis.json --grpc-address :5003 --libp2p :30303 --jsonrpc :10003 --seal --log-level DEBUG [--relayer]

./polygon-edge server --data-dir ./test-chain-4 --chain genesis.json --grpc-address :5004 --libp2p :30304 --jsonrpc :10004 --seal --log-level DEBUG [--relayer]
```