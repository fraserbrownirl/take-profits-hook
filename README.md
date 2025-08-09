# Privacy-Enhanced Take Profit Hook for Uniswap v4

## Overview

This implementation combines Zeroinch's (https://ethglobal.com/showcase/zeroinch-cfkjq) privacy features with a Uniswap v4 take profit hook, enabling users to place and execute limit orders privately using zero-knowledge proofs.

## Key Features

### 1. **Private Deposits and Notes**
- Users can deposit tokens and receive a private note (commitment) in a Merkle tree
- Notes are created using Poseidon hash functions for efficiency in ZK circuits
- Each note contains: token address, amount, and a secret hash

### 2. **Zero-Knowledge Proof System**
- Uses nullifiers to prevent double-spending
- Merkle tree with 10 levels for storing private commitments
- Maintains a history of 30 Merkle roots for flexibility

### 3. **Private Take Profit Orders**
- Place limit orders without revealing:
  - Your identity
  - Exact amounts (until execution)
  - Connection to previous transactions
- Orders execute automatically when price conditions are met

### 4. **Order Management**
- **Place**: Create orders using ZK proofs to prove ownership of funds
- **Cancel**: Cancel orders using a cancel hash (preimage)
- **Execute**: Automatic execution when tick conditions are met

## Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│                 │     │              │     │                 │
│   User Wallet   │────▶│  ZK Prover   │────▶│  Privacy Hook   │
│                 │     │              │     │                 │
└─────────────────┘     └──────────────┘     └────────┬────────┘
                                                       │
                                              ┌────────▼────────┐
                                              │                 │
                                              │  Uniswap v4     │
                                              │  Pool Manager   │
                                              │                 │
                                              └─────────────────┘
```

## How It Works

### Step 1: Private Deposit
```solidity
// User deposits tokens and gets a private note
hook.depositPrivate(tokenAddress, amount, secretHash);
```

### Step 2: Place Private Order
```solidity
// User creates a ZK proof showing they own funds
// and places a take profit order
ZKProofInput memory zkInput = ZKProofInput({
    merkleRoot: currentRoot,
    orderHash: orderHash,
    tokenIn: USDC,
    tokenOut: ETH,
    amountIn: 1000e6,
    tickToSellAt: 1000,
    nullifier: [nullifier1, nullifier2],
    // ... other fields
});

hook.placePrivateOrder(zkInput, proof);
```

### Step 3: Automatic Execution
- Hook monitors swaps in the pool
- When tick crosses the target price, order executes
- Output tokens are added as a new private note

### Step 4: Private Withdrawal
```solidity
// User can withdraw privately using ZK proof
hook.withdrawPrivate(token, amount, recipient, nullifiers, proof, merkleRoot);
```

## Privacy Guarantees

1. **Deposit Privacy**: Deposits are recorded as commitments, hiding amounts
2. **Order Privacy**: Orders don't reveal the placer's identity
3. **Execution Privacy**: Executed orders create new private notes
4. **Withdrawal Privacy**: Withdrawals use ZK proofs to maintain anonymity

## Technical Components

### Merkle Tree
- 10 levels deep (supports 1024 notes)
- Uses Poseidon hash for efficiency
- Maintains 30 historical roots

### Nullifiers
- Prevent double-spending of notes
- Each note consumption produces unique nullifiers
- Publicly visible but unlinkable to notes

### ZK Proof System
- Verifies ownership of notes without revealing which ones
- Ensures valid state transitions
- Uses Honk/Plonk-style proofs (in production, you'd use the actual Honk verifier from Zeroinch)

## Security Considerations

1. **Verifier Contract**: The example uses a simplified verifier - production needs a real ZK verifier
2. **Order Matching**: Current implementation is basic - production needs sophisticated matching
3. **Front-running**: Consider adding commit-reveal schemes for order placement
4. **Privacy Set**: Larger anonymity sets provide better privacy

## Gas Optimization

- Poseidon hash is more gas-efficient than SHA256/Keccak for ZK circuits
- Batch operations can reduce per-transaction costs
- Consider implementing a relayer system for gas abstraction

## Future Enhancements

1. **Multi-Asset Pools**: Support for multiple tokens in notes
2. **Advanced Order Types**: Stop-loss, trailing stops, etc.
3. **Cross-Pool Orders**: Execute across multiple Uniswap pools
4. **Governance Token**: Privacy-preserving governance participation
5. **Layer 2 Integration**: Deploy on L2s for lower costs

## Testing

```bash
# Run tests
forge test --match-contract PrivacyTakeProfitHookTest

# Run with verbosity
forge test --match-contract PrivacyTakeProfitHookTest -vvv
```

## Deployment

1. Deploy the verifier contract
2. Deploy the hook with verifier address
3. Initialize pools with the hook
4. Users can start making private deposits and orders

## License

MIT
