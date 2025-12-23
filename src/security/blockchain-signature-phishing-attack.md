# Signature Phishing Attack

"Signature Phishing Attack" (also known as "Permit Phishing" or "Blind Signing Attack"

How can this happen?

You may fall victim to a signature-based phishing attack if you sign a malicious request in MetaMask without realizing what you’re approving. This is different from a traditional transaction because:

1. **You signed a message, not a blockchain transaction** - That's why you don't see a transaction from your address
2. **The signature gave the attacker permission** to move your tokens
3. **The attacker executed the drain transaction** later using your signed permission

## Type of Attack: **Permit/Approval Phishing**

This attack exploits two Ethereum features:
- **ERC-20 Permit functions** (EIP-2612) - allows approvals via off-chain signatures
- **SetApprovalForAll** for NFTs - bulk token approvals

When you signed in MetaMask, you likely approved one of these:
- `permit()` - giving unlimited token spending rights
- `setApprovalForAll()` - giving control over all your NFTs
- `increaseAllowance()` - increasing spending limits

## How It Works

1. **Phishing site** tricks you into connecting wallet
2. **You sign** what looks like a login/verification in MetaMask
3. **The signature** is actually an approval/permit message
4. **Attacker uses your signature** to authorize themselves to spend your tokens
5. **Attacker drains** your wallet in a separate transaction (which appears on the blockchain from THEIR address, not yours)

## Why You Don't See Your Transaction

The **drain transaction** appears on the blockchain but:
- It's initiated by the **attacker's address**, not yours
- Your signature was used as **authorization**, not as the transaction itself

## What to Do Now

1. **Immediately revoke all approvals** at:
   - revoke.cash
   - etherscan.io/tokenapprovalchecker

2. **Create a new wallet** - Your current wallet is compromised

3. **Never reuse** that wallet again, even if you revoke approvals

4. **Report** to the phishing site host if possible

## Red Flags to Watch For

- Sites asking you to "verify" or "claim" by signing
- MetaMask showing `setApprovalForAll` or `permit` functions
- Urgent messages about limited-time offers
- Sites with URLs slightly different from legitimate ones

Unfortunately, these attacks are becoming extremely common and sophisticated. The funds are typically unrecoverable once moved, especially if sent through mixers like Tornado Cash.

## Solana (SVM - Solana Virtual Machine) is also vulnerable

**Solana (SVM - Solana Virtual Machine) is also vulnerable** to similar signature phishing attacks, though the mechanics are somewhat different from Ethereum.

## Solana/SPL Token Signature Phishing

### How It Works on Solana

On Solana, the attack typically involves:

1. **Token Account Delegation** - You sign a transaction that delegates authority over your SPL tokens
2. **Authority Transfer** - You unknowingly sign to change the authority/owner of your token accounts
3. **Approval Instructions** - Similar to ERC-20 approvals, but using Solana's `Approve` instruction for SPL tokens

### Key Differences from Ethereum

**Solana-specific vulnerabilities:**

- **Transaction simulation can be misleading** - Phantom and other wallets show transaction previews, but complex transactions with multiple instructions can hide malicious actions
- **Program Derived Addresses (PDAs)** - Attackers can create transactions where you unknowingly authorize PDAs controlled by them
- **Durable Nonces** - Allow attackers to hold signed transactions and execute them later
- **Multiple instructions in one transaction** - A legitimate-looking swap might hide a `SetAuthority` instruction

### Common Solana Phishing Patterns

1. **Fake NFT mints** - Sign to "mint" but actually transfer token account authority
2. **Fake airdrops** - "Claim" requires signing away token permissions
3. **Malicious dApp connections** - Connecting wallet + signing gives away control
4. **Token account authority changes** - `SetAuthority` instruction hidden in transaction

### What You See in Wallet

Phantom/Solflare might show:
- "Sign Transaction" (very vague)
- Multiple instructions bundled together
- Authority changes buried in technical details
- Legitimate-looking program interactions (often using verified programs as decoys)

### Protection on Solana

1. **Carefully review ALL instructions** in a transaction, not just the summary
2. **Check for `SetAuthority` instructions** - huge red flag
3. **Verify program addresses** - ensure you're interacting with the real program
4. **Use hardware wallets** - they show raw transaction data
5. **Revoke delegations** at: step.finance/revoke or revoke.cash (supports Solana)

### Example Solana Attack Flow

```
1. Visit fake NFT mint site
2. Click "Mint Free NFT"
3. Phantom shows: "Approve transaction"
4. Hidden inside: SetAuthority instruction
5. You sign → Your SPL token accounts now owned by attacker
6. Attacker drains all SPL tokens
```

### Key Technical Detail

Unlike Ethereum's off-chain signatures (EIP-712), Solana phishing usually involves:
- **Actual on-chain transactions** you sign
- But with **multiple instructions** where malicious ones are hidden
- Or **durable nonces** allowing delayed execution

So yes, **SPL tokens are absolutely vulnerable** to signature phishing, and in some ways it's even more dangerous because Solana transactions can pack multiple instructions that wallets don't always display clearly.


## Wallet Behavior Comparison

### **Phantom Wallet**
- **Shows a summary view** by default (simplified)
- **CAN show multiple instructions** but often collapses them
- Has an "Advanced" or "Details" dropdown to see all instructions
- **Problem:** Most users don't expand this section
- Recent versions have improved, but still not always clear

### **Solflare**
- **Better instruction breakdown** than Phantom
- Shows each instruction more explicitly
- Still can be confusing with complex transactions
- More technical display (good for advanced users)

### **Backpack**
- **Simulation-based display** - shows expected changes
- Focuses on "what will happen" rather than raw instructions
- Can miss hidden malicious instructions if simulation is incomplete

### **Hardware Wallets (Ledger/Trezor)**
- **Show raw transaction data** - all instructions visible
- **But very technical** - hard for average users to understand
- Most secure IF you know what you're looking at

## The Real Problem

**Most wallets show a "friendly" summary** like:
```
✅ Swap 10 SOL for 500 USDC
✅ Approve transaction
```

**But the actual transaction might contain:**
```
Instruction 1: Swap SOL → USDC ✅ (shown)
Instruction 2: SetAuthority on USDC account ⚠️ (hidden!)
Instruction 3: Transfer authority to attacker 🚨 (hidden!)
Instruction 4: Close account (hidden!)
```

### Example - What You See vs Reality

**Phantom might show:**
```
"Approve Token Swap"
Amount: 10 SOL
Expected: ~500 USDC
```

**Actual transaction contains:**
- Jupiter swap instruction (legitimate)
- SetAuthority instruction (MALICIOUS - buried in details)
- Approve instruction (MALICIOUS - gives spending rights)

## How to Check All Instructions

### In Phantom:
1. Click "View Transaction Details"
2. Expand "Instructions" section
3. Check EACH instruction individually
4. Look for keywords: `SetAuthority`, `Approve`, `Transfer`, `CloseAccount`

### In Solflare:
1. Instructions are shown more prominently
2. Still need to click through each one
3. Look at the "Program" field for each


## Red Flags to Watch For

🚨 **Multiple instructions** when you expected just one action  
🚨 **"Unknown instruction"** labels in wallet  
🚨 **Program addresses you don't recognize**  
🚨 **SetAuthority anywhere** in the transaction  
🚨 **Approve instructions** for tokens you're not actively using  
🚨 **Wallet shows "Advanced" section** you need to expand

## Recent Improvements

Phantom and other wallets have been improving:
- Better simulation displays
- More warnings for risky instructions
- Flagging of suspicious programs
- But still **not perfect**

## The Harsh Reality

**Most users never expand the details**, which is exactly what attackers count on. The "one-click" convenience of Solana transactions is also its biggest security weakness.

### Safest Approach:
1. **Always expand full transaction details**
2. **If there are multiple instructions for a "simple" action** → RED FLAG
3. **Use simulation tools** like Solscan before signing
4. **Hardware wallet** for large amounts
5. **When in doubt, reject** the transaction

**The bottom line:** Solana wallets CAN show multiple instructions, but they often don't make them prominent enough, and most users don't look. This is why Solana phishing is so effective.
