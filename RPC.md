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

## Example 2: Create "Send Eth" Script (sendEth.js)
Prerequisites
Make sure you have Node.js installed, and the ethers package set up:

```bash
npm install ethers readline-sync
```


Script: sendEth.js

javascript

```bash
const { ethers } = require('ethers');
const readlineSync = require('readline-sync');

// Initialize provider (use your RPC URL)
const provider = new ethers.JsonRpcProvider('http://localhost:8545'); // Replace with your RPC endpoint

// Function to send ETH
async function sendEth() {
    try {
        // Step 1: Get sender details
        const senderAddress = readlineSync.question('Enter the sender wallet address: ');
        const privateKey = readlineSync.question('Enter the sender private key: ', {
            hideEchoBack: true,
        });

        // Step 2: Create a Wallet instance
        const wallet = new ethers.Wallet(privateKey, provider);

        // Verify the sender address matches the private key
        if (wallet.address.toLowerCase() !== senderAddress.toLowerCase()) {
            console.log('Error: Private key does not match the sender address.');
            return;
        }

        // Step 3: Get recipient details
        const recipientAddress = readlineSync.question('Enter the recipient wallet address: ');

        // Step 4: Get amount to send
        const amount = readlineSync.question('Enter the amount of ETH to send: ');

        // Convert ETH amount to Wei (smallest denomination of ETH)
        const value = ethers.parseEther(amount);

        // Step 5: Create transaction object
        const tx = {
            to: recipientAddress,
            value: value,
            gasLimit: 21000, // Standard gas limit for simple ETH transfer
        };

        // Step 6: Send transaction
        console.log('Sending transaction...');
        const txResponse = await wallet.sendTransaction(tx);

        // Step 7: Wait for the transaction to be mined
        const receipt = await txResponse.wait();
        console.log('Transaction successful!');
        console.log(`Transaction hash: ${receipt.transactionHash}`);
    } catch (error) {
        console.error('Error:', error.message);
    }
}
```

## Run the function sendEth();

Save the script above to a file named sendEth.js.

Open your terminal and navigate to the directory where the script is located.

Run the script with:

```bash
node sendEth.js
```

Example Output
markdown
```bash
Enter the sender wallet address: 0x108B5d9A49F8275c1194D07920627A28a55bC7e4
Enter the sender private key: ***********************
Enter the recipient wallet address: 0x14086467FeC7672f82C4dBAAe12B02698979E13c
Enter the amount of ETH to send: 0.1
Sending transaction...
Transaction successful!
Transaction hash: 0x12345abcd6789ef...
```

## Notes
* Replace 'http://localhost:8545' with your actual Ethereum node RPC endpoint (such as Infura, Alchemy, or your Nethermind node).
* Ensure you have sufficient ETH in the sender's wallet to cover both the amount and gas fees.
* The private key should never be exposed. Use environment variables or secure vaults in a production environment.
