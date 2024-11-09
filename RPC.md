## Setting up an Execution RPC endpoint using Nethermind 

### Prerequisites
### Before we start, make sure you have the following:

- Nethermind installed on your system.

     Download Nethermind and follow the installation instructions.
- .NET Core Runtime (version 6.0 or higher) installed.

     Install .NET Core

Make sure you have Node.js installed, and the ethers package set up:

## Connecting via Ethers.js

Install ethers:
```bash
npm install ethers
```


  ## Verify the RPC Endpoint

  Check if the RPC endpoint is up and running by sending a simple request using curl:

```bash
  curl -X POST \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  http://localhost:8545
```


## Example 1: Create a "Get Wallet Balance" Script (getBalance.js)

Create a new JavaScript file named getBalance.js with the following content:

javascript

```bash
const { ethers } = require('ethers');

// Set your RPC endpoint URL
const RPC_URL = 'http://localhost:8545'; // Replace with your actual RPC URL if different

// Initialize a provider
const provider = new ethers.JsonRpcProvider(RPC_URL);

// The wallet address you want to check
const walletAddress = '0x108B5d9A49F8275c1194D07920627A28a55bC7e4';

async function getBalance() {
    try {
        // Get the balance in Wei (smallest unit of ETH)
        const balanceWei = await provider.getBalance(walletAddress);
        
        // Convert balance from Wei to Ether
        const balanceEth = ethers.formatEther(balanceWei);

        console.log(`Balance of wallet ${walletAddress}:`);
        console.log(`${balanceEth} ETH`);
    } catch (error) {
        console.error('Error fetching balance:', error);
    }
}

getBalance();
```

## Run the function

Run the Script
Execute the script using Node.js:

```bash
node getBalance.js
```

## Explanation
* RPC_URL: Points to your Nethermind RPC endpoint. Update this to match your setup if needed.
* ethers.JsonRpcProvider: Connects to your specified Ethereum-compatible network.
* getBalance: The function fetches the balance in Wei and converts it to Ether.
* ethers.formatEther: Converts the balance from Wei (smallest unit of ETH) to a human-readable Ether format.

Output Example
yaml
```bash
Balance of wallet 0x108B5d9A49F8275c1194D07920627A28a55bC7e4:
1.234 ETH
```
