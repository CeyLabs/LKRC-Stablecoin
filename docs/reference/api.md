# API Reference

Complete reference documentation for all LKRC stablecoin functions, events, and modifiers.

## Contract Information

- **Name**: LKRC Token
- **Symbol**: LKRC
- **Decimals**: 18
- **Solidity Version**: ^0.8.19
- **License**: MIT

## Constructor

### `constructor(uint256 _initialSupply, address initialOwner)`

Initializes the LKRC token contract with an initial supply and owner.

**Parameters:**
- `_initialSupply` (uint256): Initial token supply in whole tokens (automatically converted to wei by multiplying by 10^18)
- `initialOwner` (address): Address that will own the contract and receive the initial supply

**Example:**
```solidity
// Deploy with 1 million tokens to deployer address
LKRC lkrc = new LKRC(1000000, msg.sender);
```

**Events Emitted:**
- `Transfer(address(0), initialOwner, _initialSupply * 10**18)`
- `OwnershipTransferred(address(0), initialOwner)`

---

## ERC20 Standard Functions

### `name() → string`

Returns the name of the token.

**Returns:** `"LKRC Token"`

### `symbol() → string`

Returns the token symbol.

**Returns:** `"LKRC"`

### `decimals() → uint8`

Returns the number of decimals used for token amounts.

**Returns:** `18`

### `totalSupply() → uint256`

Returns the total token supply in wei.

**Returns:** Total supply including all minted tokens minus burned tokens.

### `balanceOf(address account) → uint256`

Returns the token balance of a specific account.

**Parameters:**
- `account` (address): Address to query the balance of

**Returns:** Token balance in wei.

### `transfer(address to, uint256 amount) → bool`

Transfers tokens from the caller to a recipient.

**Parameters:**
- `to` (address): Recipient address
- `amount` (uint256): Amount of tokens to transfer in wei

**Returns:** `true` if successful

**Modifiers:**
- `nonReentrant`: Prevents reentrancy attacks
- `whenNotPaused`: Requires contract not to be paused
- `notBlacklisted(msg.sender)`: Caller must not be blacklisted
- `notBlacklisted(to)`: Recipient must not be blacklisted

**Reverts:**
- "LKRC: account is blacklisted" - If sender or recipient is blacklisted
- "Pausable: paused" - If contract is paused
- "ERC20: transfer amount exceeds balance" - If sender has insufficient balance

**Events Emitted:**
- `Transfer(msg.sender, to, amount)`

**Example:**
```solidity
// Transfer 100 tokens
lkrc.transfer(recipientAddress, 100 * 10**18);
```

### `transferFrom(address from, address to, uint256 amount) → bool`

Transfers tokens from one address to another using the allowance mechanism.

**Parameters:**
- `from` (address): Sender address
- `to` (address): Recipient address
- `amount` (uint256): Amount of tokens to transfer in wei

**Returns:** `true` if successful

**Modifiers:**
- `nonReentrant`: Prevents reentrancy attacks
- `whenNotPaused`: Requires contract not to be paused
- `notBlacklisted(from)`: Sender must not be blacklisted
- `notBlacklisted(to)`: Recipient must not be blacklisted
- `notBlacklisted(msg.sender)`: Caller must not be blacklisted

**Reverts:**
- "LKRC: account is blacklisted" - If any party is blacklisted
- "Pausable: paused" - If contract is paused
- "ERC20: insufficient allowance" - If allowance is insufficient

**Events Emitted:**
- `Transfer(from, to, amount)`
- `Approval(from, msg.sender, newAllowance)` (if allowance updated)

### `approve(address spender, uint256 amount) → bool`

Sets the allowance for a spender to transfer tokens on behalf of the caller.

**Parameters:**
- `spender` (address): Address authorized to spend tokens
- `amount` (uint256): Maximum amount the spender can transfer in wei

**Returns:** `true` if successful

**Modifiers:**
- `nonReentrant`: Prevents reentrancy attacks
- `notBlacklisted(msg.sender)`: Caller must not be blacklisted
- `notBlacklisted(spender)`: Spender must not be blacklisted

**Reverts:**
- "LKRC: account is blacklisted" - If caller or spender is blacklisted

**Events Emitted:**
- `Approval(msg.sender, spender, amount)`

### `allowance(address owner, address spender) → uint256`

Returns the remaining number of tokens that spender is allowed to spend on behalf of owner.

**Parameters:**
- `owner` (address): Token owner address
- `spender` (address): Spender address

**Returns:** Remaining allowance in wei.

---

## Administrative Functions

### `mint(address to, uint256 amount)`

Mints new tokens to a specified address. Only callable by the owner.

**Parameters:**
- `to` (address): Recipient address
- `amount` (uint256): Amount of tokens to mint in wei

**Modifiers:**
- `onlyOwner`: Only contract owner can call
- `nonReentrant`: Prevents reentrancy attacks
- `notBlacklisted(to)`: Recipient must not be blacklisted

**Reverts:**
- "Ownable: caller is not the owner" - If caller is not the owner
- "LKRC: account is blacklisted" - If recipient is blacklisted

**Events Emitted:**
- `Transfer(address(0), to, amount)`

**Example:**
```solidity
// Mint 1000 tokens to user address
lkrc.mint(userAddress, 1000 * 10**18);
```

**Gas Cost:** ~45,000-60,000 gas

### `burn(uint256 amount)`

Burns tokens from the owner's balance. Only callable by the owner.

**Parameters:**
- `amount` (uint256): Amount of tokens to burn in wei

**Modifiers:**
- `onlyOwner`: Only contract owner can call
- `nonReentrant`: Prevents reentrancy attacks

**Reverts:**
- "Ownable: caller is not the owner" - If caller is not the owner
- "ERC20: burn amount exceeds balance" - If owner has insufficient balance

**Events Emitted:**
- `Transfer(owner(), address(0), amount)`

**Example:**
```solidity
// Burn 500 tokens from owner balance
lkrc.burn(500 * 10**18);
```

**Gas Cost:** ~30,000-45,000 gas

---

## Pause Functions

### `pause()`

Pauses all token transfers, approvals, minting, and burning. Only callable by the owner.

**Modifiers:**
- `onlyOwner`: Only contract owner can call

**Reverts:**
- "Ownable: caller is not the owner" - If caller is not the owner
- "Pausable: paused" - If already paused

**Events Emitted:**
- `Paused(msg.sender)`

**Example:**
```solidity
// Emergency pause
lkrc.pause();
```

**Gas Cost:** ~30,000 gas

### `unpause()`

Resumes normal operations after a pause. Only callable by the owner.

**Modifiers:**
- `onlyOwner`: Only contract owner can call

**Reverts:**
- "Ownable: caller is not the owner" - If caller is not the owner
- "Pausable: not paused" - If not currently paused

**Events Emitted:**
- `Unpaused(msg.sender)`

**Example:**
```solidity
// Resume operations
lkrc.unpause();
```

**Gas Cost:** ~30,000 gas

---

## Blacklist Functions

### `addToBlacklist(address account)`

Adds an address to the blacklist. Only callable by the owner.

**Parameters:**
- `account` (address): Address to blacklist

**Modifiers:**
- `onlyOwner`: Only contract owner can call

**Reverts:**
- "Ownable: caller is not the owner" - If caller is not the owner
- "LKRC: account is already blacklisted" - If address is already blacklisted

**Events Emitted:**
- `AddedToBlacklist(account)`

**Example:**
```solidity
// Blacklist a suspicious address
lkrc.addToBlacklist(suspiciousAddress);
```

**Gas Cost:** ~45,000 gas

### `removeFromBlacklist(address account)`

Removes an address from the blacklist. Only callable by the owner.

**Parameters:**
- `account` (address): Address to remove from blacklist

**Modifiers:**
- `onlyOwner`: Only contract owner can call

**Reverts:**
- "Ownable: caller is not the owner" - If caller is not the owner
- "LKRC: account is not blacklisted" - If address is not blacklisted

**Events Emitted:**
- `RemovedFromBlacklist(account)`

**Example:**
```solidity
// Remove from blacklist after review
lkrc.removeFromBlacklist(addressToRestore);
```

**Gas Cost:** ~30,000 gas

### `addToBlacklistBatch(address[] calldata accounts)`

Adds multiple addresses to the blacklist in a single transaction. Only callable by the owner.

**Parameters:**
- `accounts` (address[]): Array of addresses to blacklist

**Modifiers:**
- `onlyOwner`: Only contract owner can call

**Events Emitted:**
- `AddedToBlacklist(account)` for each newly blacklisted address

**Example:**
```solidity
// Batch blacklist multiple addresses
address[] memory addresses = new address[](3);
addresses[0] = address1;
addresses[1] = address2;
addresses[2] = address3;
lkrc.addToBlacklistBatch(addresses);
```

**Gas Cost:** ~25,000 + 20,000 per address

### `removeFromBlacklistBatch(address[] calldata accounts)`

Removes multiple addresses from the blacklist in a single transaction. Only callable by the owner.

**Parameters:**
- `accounts` (address[]): Array of addresses to remove from blacklist

**Modifiers:**
- `onlyOwner`: Only contract owner can call

**Events Emitted:**
- `RemovedFromBlacklist(account)` for each address removed

**Example:**
```solidity
// Batch remove from blacklist
address[] memory addresses = new address[](3);
addresses[0] = address1;
addresses[1] = address2;
addresses[2] = address3;
lkrc.removeFromBlacklistBatch(addresses);
```

**Gas Cost:** ~25,000 + 15,000 per address

### `isBlacklisted(address account) → bool`

Checks if an address is blacklisted.

**Parameters:**
- `account` (address): Address to check

**Returns:** `true` if blacklisted, `false` otherwise

**Example:**
```solidity
// Check blacklist status
bool isBlocked = lkrc.isBlacklisted(userAddress);
```

**Gas Cost:** ~2,500 gas (view function)

### `destroyBlackFunds(address blackListedUser)`

Burns all tokens held by a blacklisted address. Only callable by the owner.

**Parameters:**
- `blackListedUser` (address): Blacklisted address whose funds will be destroyed

**Modifiers:**
- `onlyOwner`: Only contract owner can call
- `nonReentrant`: Prevents reentrancy attacks

**Reverts:**
- "Ownable: caller is not the owner" - If caller is not the owner
- "LKRC: account is not blacklisted" - If address is not blacklisted

**Events Emitted:**
- `DestroyedBlackFunds(blackListedUser, balance)`
- `Transfer(blackListedUser, address(0), balance)`

**Example:**
```solidity
// Destroy funds from sanctioned address
lkrc.destroyBlackFunds(sanctionedAddress);
```

**Gas Cost:** ~40,000-50,000 gas

---

## Ownership Functions

### `owner() → address`

Returns the current owner of the contract.

**Returns:** Owner address

### `transferOwnership(address newOwner)`

Transfers ownership of the contract to a new address. Only callable by the current owner.

**Parameters:**
- `newOwner` (address): New owner address (cannot be zero address)

**Modifiers:**
- `onlyOwner`: Only contract owner can call

**Reverts:**
- "Ownable: caller is not the owner" - If caller is not the owner
- "Ownable: new owner is the zero address" - If newOwner is zero address

**Events Emitted:**
- `OwnershipTransferred(previousOwner, newOwner)`

**Example:**
```solidity
// Transfer to multi-sig wallet
lkrc.transferOwnership(multiSigWalletAddress);
```

### `renounceOwnership()`

Renounces ownership, leaving the contract without an owner. Only callable by the owner.

**Warning:** This will permanently disable all owner-only functions. Use with extreme caution.

**Modifiers:**
- `onlyOwner`: Only contract owner can call

**Events Emitted:**
- `OwnershipTransferred(previousOwner, address(0))`

---

## Events

### `Transfer(address indexed from, address indexed to, uint256 value)`

Emitted when tokens are transferred, including minting (from = 0) and burning (to = 0).

**Parameters:**
- `from`: Source address (0x0 for minting)
- `to`: Destination address (0x0 for burning)
- `value`: Amount transferred in wei

### `Approval(address indexed owner, address indexed spender, uint256 value)`

Emitted when an allowance is set via `approve()`.

**Parameters:**
- `owner`: Token owner address
- `spender`: Spender address
- `value`: Allowance amount in wei

### `AddedToBlacklist(address indexed account)`

Emitted when an address is added to the blacklist.

**Parameters:**
- `account`: Address that was blacklisted

### `RemovedFromBlacklist(address indexed account)`

Emitted when an address is removed from the blacklist.

**Parameters:**
- `account`: Address that was removed from blacklist

### `DestroyedBlackFunds(address indexed blackListedUser, uint256 balance)`

Emitted when funds from a blacklisted address are destroyed.

**Parameters:**
- `blackListedUser`: Address whose funds were destroyed
- `balance`: Amount of funds destroyed in wei

### `Paused(address account)`

Emitted when the contract is paused.

**Parameters:**
- `account`: Address that triggered the pause (always owner)

### `Unpaused(address account)`

Emitted when the contract is unpaused.

**Parameters:**
- `account`: Address that triggered the unpause (always owner)

### `OwnershipTransferred(address indexed previousOwner, address indexed newOwner)`

Emitted when ownership is transferred.

**Parameters:**
- `previousOwner`: Previous owner address
- `newOwner`: New owner address

---

## Modifiers

### `onlyOwner`

Restricts function access to the contract owner only.

### `whenNotPaused`

Requires the contract to not be in paused state.

### `notBlacklisted(address account)`

Requires the specified address to not be blacklisted.

### `nonReentrant`

Prevents reentrancy attacks by blocking nested calls.

---

## State Variables

### `blacklist` (mapping)

```solidity
mapping(address => bool) public blacklist
```

Public mapping that tracks blacklisted addresses. Returns `true` if address is blacklisted, `false` otherwise.

---

## Gas Optimization Notes

- Batch operations (`addToBlacklistBatch`, `removeFromBlacklistBatch`) are significantly more gas-efficient than multiple individual calls
- View functions (`balanceOf`, `allowance`, `isBlacklisted`, etc.) consume no gas when called externally
- Token amounts should be provided in wei (multiply by 10^18 for whole tokens)

---

## Integration Tips

1. Always check for blacklist status before attempting operations
2. Monitor events for compliance and operational tracking
3. Use batch operations when managing multiple addresses
4. Implement proper error handling for all revert conditions
5. Consider gas costs when performing administrative operations

For more details, see [Integration Guide](../integration/exchanges.md).
