# Off-chain Components

## 1. Core Architecture Principle
Our system leverages Midnight's native privacy features, ensuring that all critical operations (registration, voting, tallying) happen on-chain. This eliminates the need for complex off-chain computations.

## 2. Actual Off-chain Components

### 2.1 DApp Interface
```typescript
interface DAppInterface {
  // UI Components
  renderVotingInterface(): void;
  displayBallotOptions(): void;
  showTransactionStatus(): void;
}
```

### 2.2 Wallet Integration
```typescript
interface WalletConnector {
  // Basic wallet interactions
  connectWallet(): Promise<WalletConnection>;
  getConnectedWallet(): Wallet | null;
  
  // Transaction submission
  submitTransaction(tx: Transaction): Promise<TxHash>;
}
```

### 2.3 User Feedback
```typescript
interface UserFeedback {
  // Status updates
  showTransactionPending(): void;
  showTransactionSuccess(): void;
  showTransactionError(error: Error): void;
  
  // Confirmation displays
  showVoteReceipt(receipt: Receipt): void;
}
```

