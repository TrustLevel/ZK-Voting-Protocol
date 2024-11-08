# Cryptographic Specifications

## 1. Vote Commitment Scheme

### Pedersen Commitments
The system uses Pedersen Commitments as its core commitment scheme, providing essential properties for secure and private voting:

- **Formula**: `C = g^v * h^r`
  - `v`: The vote value
  - `r`: Random blinding factor
  - `g, h`: Generator points

### Key Properties

#### Perfect Hiding
- Conceals vote contents cryptographically
- Provides information-theoretic privacy
- No computational assumptions for privacy

#### Homomorphic Properties
- Enables private vote tallying
- Allows commitment aggregation
- Preserves vote privacy during counting

#### Implementation Benefits
- Efficient ZK circuit integration
- Low computational overhead
- Simple implementation structure

### Critical Properties

#### Binding Property
```plaintext
Ensures:
- Vote immutability after commitment
- No retroactive vote changes
- Cryptographic vote integrity
```

#### Hiding Property
```plaintext
Provides:
- Complete vote privacy
- No information leakage
- Future-proof confidentiality
```

## 2. Merkle Trees

### General Structure

#### Tree Components
```plaintext
1. Leaf Nodes
   - Hash of individual voter data
   - Voter credentials
   - Registration commitments

2. Intermediate Nodes
   - Hash of child node pairs
   - Binary tree structure
   - Efficient verification paths

3. Root Node
   - Single hash representing all data
   - Tamper-evident structure
   - Efficient state verification
```

### Poseidon Hash Implementation

#### Advantages for ZK-SNARKs
- Optimized for ZK circuits
- Minimal constraint generation
- Efficient proof computation

#### Technical Specifications
```plaintext
Properties:
- Field-friendly operations
- Optimized permutation
- Secure construction
```

### Tree Types and Requirements

#### 1. Voter Registration Tree

##### Purpose
- Store registration commitments
- Verify voter eligibility
- Maintain registration privacy

##### Structure
```plaintext
Configuration:
- Leaf: H(voter_commitment)
- Hash: Poseidon
- Dynamic height adaptation
```

##### Growth Management
```plaintext
Parameters:
- Initial minimum height
- Growth increment size
- Maximum expansion limit
```

#### 2. Nullifier Tree

##### Purpose
- Double-vote prevention
- Used nullifier tracking
- Privacy preservation

##### Structure
```plaintext
Configuration:
- Height: h = ⌈log₂(max_voters)⌉
- Leaf: H(nullifier)
- Type: Sparse Merkle Tree
```

##### Nullifier Generation
```rust
// Pseudocode for nullifier generation
fn generate_nullifier(
    credential: Credential,
    nonce: Nonce
) -> Nullifier {
    poseidon_hash([
        credential.to_field(),
        nonce.to_field()
    ])
}
```

### Technical Notes

#### Hash Function Selection
```plaintext
Poseidon advantages:
1. ZK-SNARK optimization
2. Efficient field operations
3. Proven security properties
4. Wide compatibility
```

#### Tree Configuration
```plaintext
Initial setup:
1. Minimum viable height
2. Growth parameters
3. Update mechanisms
4. Verification paths
```

#### Privacy Considerations
```plaintext
Protection measures:
1. Leaf privacy
2. Path confidentiality
3. Root publicity
4. Update privacy
```
