# Clear Pending Transactions on EVM Blockchains.

Occassionally you may encounter an error message `nonce too low` when you attempt sending a transaction over your EVM based chain. This can cause the sender's address to be unable to send further transactions, so it's important to understand the cause and solution for this issue.

## Why this happens
Usually this error is caused due to pending transactions. Pending transactions generally occurs due to mispriced gas. Gas cost on a blockchain is inherently dynamic and changes all the time. Due to this variance, some transactions might get picked up nodes but won't make it to the block. This happens with both underpricing and overpricing of gas.

## How to clear pending transactions.
In order to clear pending transactions, you first need to determine the first pending `nonce` of the sender's address. Let's say the last successul transaction by the sender had a `nonce` of `42`. This means the first pending `nonce` from the sender is `43` (we take the next integer as the new nonce).

The new nonce needs to be mentioned explicitly in this `ethersjs` script below.
```
const { ethers, Wallet } = require("ethers")
const fetch = (...args) =>
  import("node-fetch").then(({ default: fetch }) => fetch(...args));

async function main(){
    const address =  <>Address of stuck wallet</>
    const key =  <>Key of stuck wallet</>
    const rpc =  <>network HTTP RPC</>

    const provider = new ethers.providers.JsonRpcProvider(rpc)
    const signer = new Wallet(key,provider)
    const nonce = <>First pending nonce of sender's address</>
    console.log("Nonce:",nonce)

    // estimate GasPrice
    const data = await fetch( <>Gas station API</>); 
    const dataJson = await data.json();
    const gas = dataJson["fast"];

    const maxGasPrice = Math.trunc(gas.maxFee * 10 ** 9);
    console.log(`Gas Price: ${maxGasPrice}`)

    let tx = {
        to: address,
        value: 0,
        nonce,
        gasPrice: maxGasPrice
    }

    // estimate GasLimit
    const estimatedGas = await signer.estimateGas(tx)
    console.log(`Gas Limit:${estimatedGas}`)


    signer.sendTransaction({
        gasLimit: estimatedGas,
        ...tx
    })
    .then((txObj)=>{
        console.log('txHash',txObj.hash)
    })
}

main()
```

## How to avoid this problem
To avoid this problem we need to price gas correctly. Using a reliable and low latency `gas station API` can help mitigate this issue.