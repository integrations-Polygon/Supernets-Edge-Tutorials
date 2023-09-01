# Add 5th Local Validator to your Supernet

This tutorial assumes you have followed the instructions [here](https://github.com/integrations-Polygon/Supernets-Edge-Tutorials/blob/master/Supernets_v1.1.0_setup.md) to setup a local Supernet with 4 nodes.

In order to do so you must have opened 4 terminal windows on your machine. Now create a new, 5th window and enter the following commands:

## 1. Initialise 5th validator secrets
```
./polygon-edge polybft-secrets --data-dir test-chain-5 --insecure
```
This will return the new validator's address. Store it as a variable for further use
```
VALIDATOR_5=<New_Validator_Address_Here>
```

## 2. Set env variables for current terminal session
The variables set during the initial setup won't work in a new window, so we need to set them again
```
DEPLOYER_ADDRESS=<deployer wallet address>
DEPLOYER_KEY=<hex encoded deployer key>
ROOTCHAIN_RPC=<rootchain_rpc_here>
ROOTCHAIN_STAKE_TOKEN=<stake_token_address_here>
```

## 3. Set Stake Manager, Supernet Manager, and Supernet ID
```
STAKE_MANAGER=$(jq -r '.params.engine.polybft.bridge.stakeManagerAddr' "genesis.json")
SUPERNET_MANAGER=$(jq -r '.params.engine.polybft.bridge.customSupernetManagerAddr' "genesis.json")
SUPERNET_ID=$(jq -r '.params.engine.polybft.supernetID' "genesis.json")
```

## 4. Fund the new validator
```
./polygon-edge rootchain fund --addresses $VALIDATOR_5 --amounts 100000000000000000 --json-rpc $ROOTCHAIN_RPC --private-key $DEPLOYER_KEY
```

## 5. Whitelist Validator
```
./polygon-edge polybft whitelist-validators --private-key $DEPLOYER_KEY --addresses $VALIDATOR_5 --supernet-manager $SUPERNET_MANAGER --jsonrpc $ROOTCHAIN_RPC
```

## 6. Register Validator
```
./polygon-edge polybft register-validator --data-dir ./test-chain-5 --supernet-manager $SUPERNET_MANAGER --jsonrpc $ROOTCHAIN_RPC
``` 

## 7. Initialise Staking
Note that yo will need to transfer the required amount of `Rootchain Stake Token`s to the new validator's address on rootchain
```
./polygon-edge polybft stake --data-dir ./test-chain-5 --supernet-id $SUPERNET_ID --amount 100000000000000000 --stake-manager $STAKE_MANAGER --stake-token $ROOTCHAIN_STAKE_TOKEN --jsonrpc $ROOTCHAIN_RPC
```

## 8. Finally, start new validator
```
./polygon-edge server --data-dir ./test-chain-5 --chain genesis.json --grpc-address :5005 --libp2p :30305 --jsonrpc :10005 --seal --log-level DEBUG --relayer
```