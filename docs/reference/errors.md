# Error Reference and Troubleshooting

Complete guide to understanding and resolving errors in LKRC stablecoin operations.

## Error Categories

- [ERC20 Standard Errors](#erc20-standard-errors)
- [Access Control Errors](#access-control-errors)
- [Blacklist Errors](#blacklist-errors)
- [Pause Errors](#pause-errors)
- [Deployment Errors](#deployment-errors)
- [Integration Errors](#integration-errors)

---

## ERC20 Standard Errors

### `ERC20: transfer amount exceeds balance`

**Cause:** Attempting to transfer more tokens than the sender's balance.

**Solution:**
- Check sender's balance with `balanceOf(sender)`
- Ensure sufficient tokens before transferring
- Consider slippage and fees in DeFi integrations

**Example:**
```javascript
// Check balance first
const balance = await lkrc.balanceOf(senderAddress);
if (balance >= amount) {
  await lkrc.transfer(recipient, amount);
}
```

### `ERC20: insufficient allowance`

**Cause:** Attempting `transferFrom` without sufficient allowance.

**Solution:**
- Increase allowance with `approve(spender, amount)`
- Check current allowance with `allowance(owner, spender)`
- Request user approval in dApp interfaces

**Example:**
```javascript
// Check and set allowance
const currentAllowance = await lkrc.allowance(owner, spender);
if (currentAllowance < amount) {
  await lkrc.approve(spender, amount);
}
await lkrc.transferFrom(owner, recipient, amount);
```

### `ERC20: approve from the zero address`

**Cause:** Attempting to call `approve` from the zero address (0x0).

**Solution:**
- Ensure the caller address is valid and not zero
- Check wallet connection in frontend applications

### `ERC20: transfer to the zero address`

**Cause:** Attempting to transfer tokens to the zero address (0x0).

**Solution:**
- Validate recipient address before transferring
- Implement address validation in your application

**Example:**
```javascript
// Validate address
if (recipientAddress === ethers.ZeroAddress) {
  throw new Error("Cannot transfer to zero address");
}
```

### `ERC20: burn amount exceeds balance`

**Cause:** Attempting to burn more tokens than the owner's balance.

**Solution:**
- Check owner's balance before burning
- Ensure sufficient tokens are available

---

## Access Control Errors

### `Ownable: caller is not the owner`

**Cause:** Attempting to call an owner-only function from a non-owner address.

**Affected Functions:**
- `mint()`
- `burn()`
- `pause()` / `unpause()`
- `addToBlacklist()` / `removeFromBlacklist()`
- `addToBlacklistBatch()` / `removeFromBlacklistBatch()`
- `destroyBlackFunds()`
- `transferOwnership()`

**Solution:**
- Verify you're using the correct owner wallet
- Check current owner with `owner()`
- Ensure ownership hasn't been transferred

**Example:**
```javascript
// Check ownership
const currentOwner = await lkrc.owner();
console.log("Current owner:", currentOwner);
console.log("Caller address:", await signer.getAddress());
```

### `Ownable: new owner is the zero address`

**Cause:** Attempting to transfer ownership to the zero address (0x0).

**Solution:**
- Provide a valid non-zero address
- Use multi-sig wallets for production ownership

---

## Blacklist Errors

### `LKRC: account is blacklisted`

**Cause:** Attempting an operation involving a blacklisted address.

**Affected Operations:**
- Transfers from/to blacklisted addresses
- Approvals involving blacklisted addresses
- Minting to blacklisted addresses

**Solution:**
- Check blacklist status with `isBlacklisted(address)`
- Remove from blacklist if appropriate: `removeFromBlacklist(address)`
- Wait for compliance review before retry

**Example:**
```javascript
// Check before transfer
const isBlocked = await lkrc.isBlacklisted(recipientAddress);
if (isBlocked) {
  console.error("Recipient is blacklisted");
  return;
}
await lkrc.transfer(recipientAddress, amount);
```

### `LKRC: account is already blacklisted`

**Cause:** Attempting to add an already blacklisted address to the blacklist.

**Solution:**
- Check blacklist status before adding
- Use `addToBlacklistBatch()` which skips already blacklisted addresses
- No action needed if address is already blocked

### `LKRC: account is not blacklisted`

**Cause:** Attempting to remove an address that isn't blacklisted, or destroy funds from a non-blacklisted address.

**Solution:**
- Verify blacklist status with `isBlacklisted(address)`
- Add to blacklist before calling `destroyBlackFunds()`

---

## Pause Errors

### `Pausable: paused`

**Cause:** Attempting token operations while the contract is paused.

**Affected Operations:**
- `transfer()`
- `transferFrom()`
- All token movements

**Solution:**
- Wait for contract to be unpaused
- Contact contract owner/administrator
- Monitor `Unpaused` event
- Check pause status in your application before operations

**Example:**
```javascript
// Check pause status
const isPaused = await lkrc.paused();
if (isPaused) {
  console.log("Contract is currently paused");
  return;
}
```

### `Pausable: not paused`

**Cause:** Attempting to unpause a contract that isn't paused.

**Solution:**
- Check pause status before calling `unpause()`
- No action needed if contract is already operational

---

## Deployment Errors

### `insufficient funds for gas`

**Cause:** Deployer wallet doesn't have enough ETH to pay for deployment gas.

**Solution:**
- Add ETH to deployer wallet
- Estimate gas costs before deployment
- Use testnet for testing deployments

**Gas Estimates:**
- Deployment: ~1,500,000 gas
- At 30 gwei: ~0.045 ETH
- At 100 gwei: ~0.15 ETH

### `nonce too low`

**Cause:** Transaction nonce conflicts with pending transactions.

**Solution:**
- Wait for pending transactions to complete
- Reset nonce in wallet
- Use proper nonce management in scripts

### `replacement transaction underpriced`

**Cause:** Attempting to replace a transaction with insufficient gas price increase.

**Solution:**
- Increase gas price by at least 10-20%
- Wait for original transaction to complete
- Cancel with higher gas price

---

## Integration Errors

### `reverted with reason string 'ReentrancyGuard: reentrant call'`

**Cause:** Reentrancy attack attempt detected.

**Solution:**
- This is a security feature working correctly
- Avoid nested calls to LKRC functions
- Review integration logic for unintended reentrancy

### `transaction failed` (no specific error)

**Possible Causes:**
1. Out of gas
2. Silent revert in external contract
3. Network issues

**Solution:**
- Increase gas limit
- Check transaction on block explorer
- Verify all addresses are correct
- Test on testnet first

### `execution reverted`

**General Cause:** Transaction would fail one of the contract's require statements.

**Debugging Steps:**
1. Enable detailed error messages in your provider
2. Use Tenderly or similar tools to debug
3. Test transactions with eth_call before sending
4. Check all preconditions manually

**Example:**
```javascript
// Test transaction before sending
try {
  await lkrc.transfer.staticCall(recipient, amount);
  // If no error, proceed with actual transaction
  await lkrc.transfer(recipient, amount);
} catch (error) {
  console.error("Transaction would fail:", error.reason);
}
```

---

## Common Scenarios and Solutions

### Scenario 1: Transfer Fails After Pause

**Problem:** Users cannot transfer tokens after emergency pause.

**Solution:**
```javascript
// 1. Owner unpauses the contract
await lkrc.unpause();

// 2. Wait for transaction confirmation
await tx.wait();

// 3. Retry transfer
await lkrc.transfer(recipient, amount);
```

### Scenario 2: Cannot Mint to User

**Problem:** Mint transaction reverts.

**Debugging Checklist:**
- [ ] Is caller the owner?
- [ ] Is recipient address valid (not zero)?
- [ ] Is recipient blacklisted?
- [ ] Is contract paused?

**Solution:**
```javascript
// Check all conditions
const isOwner = (await lkrc.owner()) === signerAddress;
const isPaused = await lkrc.paused();
const isBlacklisted = await lkrc.isBlacklisted(recipient);

if (!isOwner) {
  console.error("Not the owner");
} else if (isPaused) {
  await lkrc.unpause();
} else if (isBlacklisted) {
  await lkrc.removeFromBlacklist(recipient);
}

// Now mint
await lkrc.mint(recipient, amount);
```

### Scenario 3: Batch Blacklist Fails

**Problem:** `addToBlacklistBatch()` runs out of gas.

**Solution:**
- Reduce batch size (recommend max 100 addresses per batch)
- Increase gas limit
- Split into multiple transactions

**Example:**
```javascript
// Split large array into batches
const batchSize = 50;
for (let i = 0; i < addresses.length; i += batchSize) {
  const batch = addresses.slice(i, i + batchSize);
  await lkrc.addToBlacklistBatch(batch);
  console.log(`Processed batch ${i / batchSize + 1}`);
}
```

### Scenario 4: Frontend Integration Issues

**Problem:** Transactions fail in web3 frontend but work in scripts.

**Common Causes:**
- Incorrect wallet connection
- Wrong network selection
- Stale state in frontend
- Missing user approvals

**Solution:**
```javascript
// Comprehensive frontend check
async function safeTransfer(recipient, amount) {
  // 1. Check wallet connection
  if (!window.ethereum) {
    throw new Error("No wallet detected");
  }

  // 2. Check network
  const chainId = await window.ethereum.request({
    method: 'eth_chainId'
  });
  if (chainId !== '0x1') { // Mainnet
    throw new Error("Wrong network");
  }

  // 3. Check balance
  const balance = await lkrc.balanceOf(signer.address);
  if (balance < amount) {
    throw new Error("Insufficient balance");
  }

  // 4. Check blacklist
  const isBlacklisted = await lkrc.isBlacklisted(recipient);
  if (isBlacklisted) {
    throw new Error("Recipient is blacklisted");
  }

  // 5. Check pause status
  const isPaused = await lkrc.paused();
  if (isPaused) {
    throw new Error("Contract is paused");
  }

  // 6. Execute transfer
  const tx = await lkrc.transfer(recipient, amount);
  return tx.wait();
}
```

---

## Diagnostic Tools

### Check Contract State

```javascript
// Get comprehensive contract state
async function diagnoseContract() {
  const owner = await lkrc.owner();
  const isPaused = await lkrc.paused();
  const totalSupply = await lkrc.totalSupply();

  console.log("Contract Diagnostics:");
  console.log("Owner:", owner);
  console.log("Paused:", isPaused);
  console.log("Total Supply:", ethers.formatEther(totalSupply));
}
```

### Check User State

```javascript
// Get user-specific information
async function diagnoseUser(userAddress) {
  const balance = await lkrc.balanceOf(userAddress);
  const isBlacklisted = await lkrc.isBlacklisted(userAddress);

  console.log("User Diagnostics:");
  console.log("Address:", userAddress);
  console.log("Balance:", ethers.formatEther(balance));
  console.log("Blacklisted:", isBlacklisted);
}
```

### Monitor Events

```javascript
// Listen for important events
lkrc.on("Paused", (account) => {
  console.log("Contract paused by:", account);
});

lkrc.on("AddedToBlacklist", (account) => {
  console.log("Address blacklisted:", account);
});

lkrc.on("Transfer", (from, to, amount) => {
  console.log(`Transfer: ${from} -> ${to}: ${ethers.formatEther(amount)}`);
});
```

---

## Getting Help

If you encounter errors not covered in this guide:

1. Check transaction on block explorer (Etherscan) for detailed error messages
2. Review the [API Reference](api.md) for function requirements
3. Verify all preconditions are met
4. Test on testnet with same parameters
5. Check recent events for state changes
6. Contact support with transaction hash and error details

## See Also

- [API Reference](api.md) - Complete function documentation
- [Operations Guide](../operations/token-lifecycle.md) - Operational procedures
- [Security Controls](../compliance/security-controls.md) - Security features
- [Integration Guide](../integration/exchanges.md) - Integration best practices
