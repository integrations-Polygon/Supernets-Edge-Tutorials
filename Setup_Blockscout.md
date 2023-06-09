# Setup Blockscout with docker-compose on Ubuntu 22.04

## Prequisites:
- [Docker](https://docs.docker.com/engine/install/ubuntu/)

## Step 1: Clone Blockscout
Clone the official Blockscout repository and switch to your preferred release. In this tutorial we will use release v5.0.0
```
git clone 
cd blockscout
git switch -b v5.0.0 e7557c42935ce1af30861a4c1b6d004ae883f1bc
```
## Step 2: Update env variables
Navigate to the `docker-compose/envs` directory and update the env variables for the `blockscout` container
```
cd docker-compose/envs
sudo vim common-blockscout.env
```
Inside this file update the `ETHEREUM_JSONRPC_HTTP_URL` and `ETHEREUM_JSONRPC_TRACE_URL` variables to your Edge JSON RPC url.

## Step 3: Build and Run
Navigate back to the `docker-compose` directory and run the enter commands:   
```
sudo docker-compose build
sudo docker-compose up -d
```
This will run the containers in a detached mode.

If your setup was correct you should can find the Blockscout UI running on `http://localhost:4000/` by default.