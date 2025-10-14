# Getting Started with LKRC Stablecoin

This guide will help you set up, compile, test, and deploy the LKRC stablecoin contract.

## Prerequisites

- Node.js v16 or later
- npm or yarn package manager
- An Ethereum wallet with private key (for deployment)
- ETH for gas fees (testnet or mainnet)

## Installation

Clone the repository and install dependencies:

```bash
git clone <repository-url>
cd LKRC-Stablecoin
npm install
```

Or if using bun:

```bash
bun install
```

## Project Structure

```
LKRC-Stablecoin/
├── LKRC.sol              # Main token contract
├── docs/                 # Documentation
├── package.json          # Project dependencies
└── README.md            # Project overview
```

## Compilation

Install Hardhat if not already installed:

```bash
npm install --save-dev hardhat
```

Compile the contract:

```bash
npx hardhat compile
```

Expected output:
```
Compiled 1 Solidity file successfully
```

## Testing

Create a test file in `test/LKRC.test.js`:

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("LKRC Token", function () {
  let lkrc, owner, addr1, addr2;

  beforeEach(async function () {
    [owner, addr1, addr2] = await ethers.getSigners();
    const LKRC = await ethers.getContractFactory("LKRC");
    lkrc = await LKRC.deploy(1000000, owner.address);
  });

  it("Should mint initial supply to owner", async function () {
    const balance = await lkrc.balanceOf(owner.address);
    expect(balance).to.equal(ethers.parseEther("1000000"));
  });
});
```

Run tests:

```bash
npx hardhat test
```

## Deployment

### Local Development (Hardhat Network)

Create a deployment script `scripts/deploy.js`:

```javascript
const { ethers } = require("hardhat");

async function main() {
  const [deployer] = await ethers.getSigners();

  console.log("Deploying LKRC with account:", deployer.address);

  const initialSupply = 1000000; // 1 million tokens
  const LKRC = await ethers.getContractFactory("LKRC");
  const lkrc = await LKRC.deploy(initialSupply, deployer.address);

  await lkrc.waitForDeployment();

  console.log("LKRC deployed to:", await lkrc.getAddress());
  console.log("Initial supply:", initialSupply, "tokens");
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Deploy locally:

```bash
npx hardhat run scripts/deploy.js
```

### Testnet Deployment (Sepolia)

1. Create a `.env` file:

```bash
PRIVATE_KEY=your_wallet_private_key
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/YOUR_INFURA_KEY
ETHERSCAN_API_KEY=your_etherscan_api_key
```

2. Update `hardhat.config.js`:

```javascript
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.19",
  networks: {
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL,
      accounts: [process.env.PRIVATE_KEY]
    }
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY
  }
};
```

3. Deploy to Sepolia:

```bash
npx hardhat run scripts/deploy.js --network sepolia
```

4. Verify on Etherscan:

```bash
npx hardhat verify --network sepolia DEPLOYED_CONTRACT_ADDRESS 1000000 OWNER_ADDRESS
```

## Post-Deployment Setup

After deployment, the owner should:

1. **Secure the owner wallet**: Transfer ownership to a multi-signature wallet for production.
2. **Test basic operations**: Verify mint, burn, pause, and blacklist functions work as expected.
3. **Set up monitoring**: Monitor emitted events for compliance and operational purposes.
4. **Document deployment**: Record contract address, network, and initial parameters.

### Transferring Ownership (Recommended for Production)

```javascript
// Transfer to multi-sig wallet
await lkrc.transferOwnership(MULTISIG_WALLET_ADDRESS);
```

## Initial Operations

### Minting Additional Tokens

```javascript
const amount = ethers.parseEther("1000"); // 1000 tokens
await lkrc.mint(recipientAddress, amount);
```

### Pausing in Emergency

```javascript
await lkrc.pause();
// Later, when resolved:
await lkrc.unpause();
```

### Managing Blacklist

```javascript
// Add single address
await lkrc.addToBlacklist(suspiciousAddress);

// Add multiple addresses (more gas efficient)
await lkrc.addToBlacklistBatch([address1, address2, address3]);
```

## Next Steps

- Read [Architecture: Contract Components](architecture/contract-components.md) to understand the contract structure
- Review [Operations: Token Lifecycle](operations/token-lifecycle.md) for operational procedures
- Check [Compliance: Security Controls](compliance/security-controls.md) for security best practices
- Explore [API Reference](reference/api.md) for complete function documentation

## Common Issues

### Issue: "insufficient funds for gas"
**Solution**: Ensure your deployment wallet has enough ETH for gas fees.

### Issue: "Ownable: caller is not the owner"
**Solution**: Administrative functions can only be called by the owner wallet.

### Issue: "LKRC: account is blacklisted"
**Solution**: Remove the address from the blacklist before attempting transfers.

For more troubleshooting help, see [Error Reference](reference/errors.md).
