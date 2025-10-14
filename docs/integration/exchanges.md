# Exchange and dApp Integration Guide

Complete guide for integrating LKRC stablecoin into exchanges, wallets, and decentralized applications.

## Overview

LKRC is a fully ERC20-compliant stablecoin with additional compliance features:
- Standard ERC20 interface for deposits and withdrawals
- Pause mechanism for emergency circuit breaking
- Blacklist functionality for regulatory compliance
- Reentrancy protection for secure integrations

## Quick Integration Checklist

- [ ] Add LKRC token contract address to system
- [ ] Implement standard ERC20 methods (transfer, transferFrom, balanceOf)
- [ ] Monitor pause events for circuit breaker status
- [ ] Monitor blacklist events for compliance
- [ ] Handle blacklist-related transfer failures
- [ ] Set up event monitoring for deposits/withdrawals
- [ ] Test on testnet before mainnet integration
- [ ] Implement proper error handling

---

## Contract Details

### Mainnet Deployment
```
Contract Address: [TO BE ADDED AFTER DEPLOYMENT]
Network: Ethereum Mainnet
Symbol: LKRC
Decimals: 18
```

### Testnet Deployment (Sepolia)
```
Contract Address: [TO BE ADDED AFTER DEPLOYMENT]
Network: Sepolia Testnet
Symbol: LKRC
Decimals: 18
```

---

## Standard ERC20 Integration

### Basic Token Operations

```javascript
const { ethers } = require("ethers");

// Contract ABI (minimal for integration)
const LKRC_ABI = [
  "function name() view returns (string)",
  "function symbol() view returns (string)",
  "function decimals() view returns (uint8)",
  "function totalSupply() view returns (uint256)",
  "function balanceOf(address account) view returns (uint256)",
  "function transfer(address to, uint256 amount) returns (bool)",
  "function transferFrom(address from, address to, uint256 amount) returns (bool)",
  "function approve(address spender, uint256 amount) returns (bool)",
  "function allowance(address owner, address spender) view returns (uint256)",
  "function isBlacklisted(address account) view returns (bool)",
  "function paused() view returns (bool)",
  "event Transfer(address indexed from, address indexed to, uint256 value)",
  "event Approval(address indexed owner, address indexed spender, uint256 value)",
  "event AddedToBlacklist(address indexed account)",
  "event RemovedFromBlacklist(address indexed account)",
  "event Paused(address account)",
  "event Unpaused(address account)"
];

// Connect to contract
const provider = new ethers.JsonRpcProvider("YOUR_RPC_URL");
const lkrc = new ethers.Contract(LKRC_ADDRESS, LKRC_ABI, provider);
```

### Checking Balances

```javascript
async function getUserBalance(userAddress) {
  try {
    const balance = await lkrc.balanceOf(userAddress);
    const formattedBalance = ethers.formatEther(balance);
    return formattedBalance;
  } catch (error) {
    console.error("Error fetching balance:", error);
    throw error;
  }
}
```

### Processing Deposits

```javascript
async function monitorDeposits(exchangeDepositAddress) {
  // Listen for incoming transfers
  lkrc.on("Transfer", async (from, to, amount, event) => {
    if (to.toLowerCase() === exchangeDepositAddress.toLowerCase()) {
      console.log("Deposit detected:");
      console.log("From:", from);
      console.log("Amount:", ethers.formatEther(amount), "LKRC");
      console.log("Transaction:", event.log.transactionHash);
      console.log("Block:", event.log.blockNumber);

      // Wait for confirmations
      const confirmations = await event.log.getBlock().then(block =>
        block.number
      );

      if (confirmations >= 12) { // Recommended: 12 confirmations
        await creditUserAccount(from, amount);
      }
    }
  });
}

async function creditUserAccount(userAddress, amount) {
  // Your internal logic to credit user's exchange balance
  console.log(`Credited ${ethers.formatEther(amount)} LKRC to user ${userAddress}`);
}
```

### Processing Withdrawals

```javascript
async function processWithdrawal(userAddress, amount) {
  try {
    // 1. Check if contract is paused
    const isPaused = await lkrc.paused();
    if (isPaused) {
      throw new Error("LKRC contract is currently paused");
    }

    // 2. Check if recipient is blacklisted
    const isBlacklisted = await lkrc.isBlacklisted(userAddress);
    if (isBlacklisted) {
      throw new Error("Recipient address is blacklisted");
    }

    // 3. Check exchange hot wallet balance
    const balance = await lkrc.balanceOf(EXCHANGE_HOT_WALLET);
    if (balance < amount) {
      throw new Error("Insufficient hot wallet balance");
    }

    // 4. Execute transfer
    const signer = new ethers.Wallet(PRIVATE_KEY, provider);
    const lkrcWithSigner = lkrc.connect(signer);

    const tx = await lkrcWithSigner.transfer(userAddress, amount);
    console.log("Withdrawal transaction submitted:", tx.hash);

    // 5. Wait for confirmation
    const receipt = await tx.wait();
    console.log("Withdrawal confirmed in block:", receipt.blockNumber);

    return receipt;
  } catch (error) {
    console.error("Withdrawal failed:", error);

    // Handle specific errors
    if (error.message.includes("blacklisted")) {
      // Notify compliance team
      await notifyCompliance(userAddress, "blacklist_withdrawal_attempt");
    }

    throw error;
  }
}
```

---

## LKRC-Specific Features

### 1. Pause Monitoring

The contract can be paused by the owner during emergencies. All transfers will fail while paused.

```javascript
async function checkPauseStatus() {
  const isPaused = await lkrc.paused();
  return isPaused;
}

// Monitor pause events
lkrc.on("Paused", (account) => {
  console.log("⚠️ LKRC contract paused by:", account);
  // Disable deposits/withdrawals in your system
  disableLKRCOperations();
  notifyAdministrators("LKRC contract paused");
});

lkrc.on("Unpaused", (account) => {
  console.log("✅ LKRC contract unpaused by:", account);
  // Re-enable deposits/withdrawals
  enableLKRCOperations();
  notifyAdministrators("LKRC contract resumed");
});
```

### 2. Blacklist Monitoring

Addresses can be blacklisted for compliance reasons. Monitor blacklist events to prevent user confusion.

```javascript
// Check if address is blacklisted before operations
async function validateAddress(address) {
  const isBlacklisted = await lkrc.isBlacklisted(address);
  if (isBlacklisted) {
    throw new Error(`Address ${address} is blacklisted`);
  }
  return true;
}

// Monitor blacklist events
lkrc.on("AddedToBlacklist", async (account) => {
  console.log("Address blacklisted:", account);

  // Check if any of your users/wallets are affected
  await handleBlacklistedAddress(account);
});

lkrc.on("RemovedFromBlacklist", (account) => {
  console.log("Address removed from blacklist:", account);
  // Re-enable operations for this address if needed
});

async function handleBlacklistedAddress(address) {
  // Freeze withdrawals to this address
  // Check if this is a user address
  const user = await findUserByAddress(address);
  if (user) {
    await freezeUserLKRCWithdrawals(user.id);
    await notifyUserOfBlacklist(user.id);
  }

  // Check if this is one of your hot/cold wallets
  if (isExchangeWallet(address)) {
    await notifySecurityTeam(`Exchange wallet ${address} was blacklisted`);
  }
}
```

### 3. Safe Transfer Implementation

Always implement proper error handling and pre-checks:

```javascript
async function safeTransfer(to, amount) {
  // Pre-flight checks
  const checks = {
    isPaused: await lkrc.paused(),
    isRecipientBlacklisted: await lkrc.isBlacklisted(to),
    isSenderBlacklisted: await lkrc.isBlacklisted(EXCHANGE_HOT_WALLET),
    balance: await lkrc.balanceOf(EXCHANGE_HOT_WALLET)
  };

  // Validate all conditions
  if (checks.isPaused) {
    throw new Error("Contract is paused");
  }
  if (checks.isRecipientBlacklisted) {
    throw new Error("Recipient is blacklisted");
  }
  if (checks.isSenderBlacklisted) {
    throw new Error("Sender wallet is blacklisted - contact support immediately");
  }
  if (checks.balance < amount) {
    throw new Error("Insufficient balance");
  }

  // Use staticCall to simulate transaction
  try {
    await lkrc.transfer.staticCall(to, amount);
  } catch (error) {
    throw new Error(`Transaction would fail: ${error.message}`);
  }

  // Execute actual transaction
  const signer = new ethers.Wallet(PRIVATE_KEY, provider);
  const tx = await lkrc.connect(signer).transfer(to, amount);

  return tx.wait();
}
```

---

## Event Monitoring Setup

### Comprehensive Event Listener

```javascript
class LKRCEventMonitor {
  constructor(contractAddress, provider) {
    this.lkrc = new ethers.Contract(contractAddress, LKRC_ABI, provider);
    this.setupListeners();
  }

  setupListeners() {
    // Transfer events (deposits/withdrawals)
    this.lkrc.on("Transfer", this.handleTransfer.bind(this));

    // Pause events
    this.lkrc.on("Paused", this.handlePaused.bind(this));
    this.lkrc.on("Unpaused", this.handleUnpaused.bind(this));

    // Blacklist events
    this.lkrc.on("AddedToBlacklist", this.handleBlacklisted.bind(this));
    this.lkrc.on("RemovedFromBlacklist", this.handleUnblacklisted.bind(this));
  }

  async handleTransfer(from, to, amount, event) {
    console.log(`Transfer: ${from} -> ${to}: ${ethers.formatEther(amount)} LKRC`);
    // Process deposit or withdrawal
    if (this.isExchangeAddress(to)) {
      await this.processDeposit(from, amount, event);
    }
  }

  handlePaused(account) {
    console.log("⚠️ Contract paused");
    this.pauseLKRCOperations();
  }

  handleUnpaused(account) {
    console.log("✅ Contract unpaused");
    this.resumeLKRCOperations();
  }

  async handleBlacklisted(account) {
    console.log("🚫 Address blacklisted:", account);
    await this.freezeAddressOperations(account);
  }

  handleUnblacklisted(account) {
    console.log("✅ Address removed from blacklist:", account);
    this.unfreezeAddressOperations(account);
  }

  pauseLKRCOperations() {
    // Disable LKRC deposits and withdrawals
  }

  resumeLKRCOperations() {
    // Re-enable LKRC deposits and withdrawals
  }

  isExchangeAddress(address) {
    // Check if address belongs to your exchange
    return EXCHANGE_ADDRESSES.includes(address.toLowerCase());
  }

  async processDeposit(from, amount, event) {
    // Your deposit processing logic
  }

  async freezeAddressOperations(address) {
    // Freeze operations for blacklisted address
  }

  unfreezeAddressOperations(address) {
    // Unfreeze operations for address
  }
}

// Initialize monitor
const monitor = new LKRCEventMonitor(LKRC_ADDRESS, provider);
```

### Historical Event Scanning

Scan for historical events on first integration:

```javascript
async function scanHistoricalEvents(fromBlock, toBlock) {
  // Get all transfers to exchange addresses
  const transferFilter = lkrc.filters.Transfer(null, EXCHANGE_DEPOSIT_ADDRESS);
  const transfers = await lkrc.queryFilter(transferFilter, fromBlock, toBlock);

  console.log(`Found ${transfers.length} deposits`);

  for (const event of transfers) {
    const { from, to, amount } = event.args;
    console.log(`Block ${event.blockNumber}: ${from} -> ${ethers.formatEther(amount)} LKRC`);
    // Process historical deposit
  }
}
```

---

## Gas Optimization

### Batch Operations

If you need to process multiple withdrawals:

```javascript
async function batchWithdrawals(withdrawals) {
  // Group by gas price tolerance
  const gasPrice = await provider.getFeeData();

  for (const withdrawal of withdrawals) {
    try {
      const tx = await processWithdrawal(
        withdrawal.address,
        withdrawal.amount
      );

      console.log(`Processed withdrawal: ${tx.hash}`);

      // Add delay to avoid nonce issues
      await new Promise(resolve => setTimeout(resolve, 1000));
    } catch (error) {
      console.error(`Failed withdrawal for ${withdrawal.address}:`, error);
      // Queue for retry
    }
  }
}
```

### Gas Estimates

Typical gas costs for LKRC operations:

| Operation | Estimated Gas | Notes |
|-----------|--------------|-------|
| `transfer()` | 45,000-60,000 | Includes blacklist and pause checks |
| `transferFrom()` | 50,000-65,000 | Includes allowance update |
| `approve()` | 30,000-45,000 | Sets allowance |
| `balanceOf()` | 2,500 | View function (free for off-chain calls) |
| `isBlacklisted()` | 2,500 | View function (free for off-chain calls) |

---

## Security Best Practices

### 1. Hot Wallet Management

```javascript
// Monitor hot wallet balance
async function monitorHotWalletBalance() {
  const balance = await lkrc.balanceOf(EXCHANGE_HOT_WALLET);
  const threshold = ethers.parseEther("100000"); // 100k LKRC

  if (balance < threshold) {
    await notifyTreasury("Hot wallet balance low");
    await transferFromColdWallet(EXCHANGE_COLD_WALLET, EXCHANGE_HOT_WALLET);
  }
}

setInterval(monitorHotWalletBalance, 3600000); // Check hourly
```

### 2. Multi-Signature Confirmations

For large withdrawals:

```javascript
async function processLargeWithdrawal(address, amount) {
  const threshold = ethers.parseEther("50000"); // 50k LKRC

  if (amount > threshold) {
    // Require multiple approvals
    const approved = await requestMultiSigApproval({
      type: "withdrawal",
      address,
      amount: ethers.formatEther(amount)
    });

    if (!approved) {
      throw new Error("Multi-sig approval required");
    }
  }

  return processWithdrawal(address, amount);
}
```

### 3. Rate Limiting

```javascript
class WithdrawalRateLimiter {
  constructor() {
    this.withdrawals = new Map();
  }

  async checkRateLimit(address, amount) {
    const key = address.toLowerCase();
    const now = Date.now();
    const windowMs = 3600000; // 1 hour
    const maxAmount = ethers.parseEther("10000"); // 10k LKRC per hour

    if (!this.withdrawals.has(key)) {
      this.withdrawals.set(key, []);
    }

    const recentWithdrawals = this.withdrawals
      .get(key)
      .filter(w => now - w.timestamp < windowMs);

    const totalAmount = recentWithdrawals.reduce(
      (sum, w) => sum + w.amount,
      BigInt(0)
    );

    if (totalAmount + BigInt(amount) > maxAmount) {
      throw new Error("Rate limit exceeded");
    }

    recentWithdrawals.push({ timestamp: now, amount: BigInt(amount) });
    this.withdrawals.set(key, recentWithdrawals);
  }
}
```

---

## Testing Checklist

Before going live:

- [ ] Test deposits on testnet
- [ ] Test withdrawals on testnet
- [ ] Test blacklist scenario (address becomes blacklisted mid-deposit)
- [ ] Test pause scenario (contract paused during withdrawal)
- [ ] Test insufficient balance scenario
- [ ] Test event monitoring and notifications
- [ ] Test hot wallet refill from cold wallet
- [ ] Load test with multiple concurrent operations
- [ ] Verify gas estimates are accurate
- [ ] Test failure recovery and retry logic

---

## Support and Resources

- [API Reference](../reference/api.md) - Complete function documentation
- [Error Reference](../reference/errors.md) - Troubleshooting guide
- [Security Controls](../compliance/security-controls.md) - Compliance features
- [Operations Guide](../operations/token-lifecycle.md) - Operational procedures

For integration support, please contact the LKRC team with:
- Contract address
- Network (mainnet/testnet)
- Integration type (exchange/wallet/dApp)
- Specific questions or issues
