# Technical Implementation Plan: On-Chain ZK Voting Components

## 1. Smart Contract Architecture

### Core Types and Data Structures
```gleam
// Protocol States
type ProtocolPhase {
  Registration
  Voting
  Tallying
  Completed
}

// Core Data Types
type BallotConfig {
  ballot_id: ByteArray
  start_time: Time
  end_time: Time
  min_participants: Int
  options: List<ByteArray>
}

type VotingState {
  phase: ProtocolPhase
  merkle_root: ByteArray
  nullifier_set: OrderedSet<ByteArray>
  vote_commitments: List<ByteArray>
  config: BallotConfig
}

// Actions
type Action {
  Register { 
    proof: ZKProof,
    commitment: ByteArray,
    nullifier: ByteArray 
  }
  Vote { 
    proof: ZKProof,
    commitment: ByteArray,
    nullifier: ByteArray 
  }
  Tally { 
    proof: ZKProof,
    result: ByteArray 
  }
}
```

### Validator Structure
```gleam
validators {
  // Registration Validator
  fn registration(
    datum: VotingState,
    redeemer: Action,
    context: ScriptContext,
  ) -> Bool {
    when redeemer is {
      Action.Register { proof, commitment, nullifier } -> {
        verify_registration(datum, proof, commitment, nullifier, context)
      }
      _ -> False
    }
  }

  // Voting Validator
  fn voting(
    datum: VotingState,
    redeemer: Action,
    context: ScriptContext,
  ) -> Bool {
    when redeemer is {
      Action.Vote { proof, commitment, nullifier } -> {
        verify_vote(datum, proof, commitment, nullifier, context)
      }
      _ -> False
    }
  }

  // Tallying Validator
  fn tallying(
    datum: VotingState,
    redeemer: Action,
    context: ScriptContext,
  ) -> Bool {
    when redeemer is {
      Action.Tally { proof, result } -> {
        verify_tally(datum, proof, result, context)
      }
      _ -> False
    }
  }
}
```

## 2. State Management Implementation

### Registration State Management
```gleam
fn verify_registration(
  state: VotingState,
  proof: ZKProof,
  commitment: ByteArray,
  nullifier: ByteArray,
  context: ScriptContext,
) -> Bool {
  // Verify phase
  verify_phase(state.phase, Registration) &&
  // Verify proof
  verify_zk_proof(proof, commitment, nullifier) &&
  // Verify nullifier not used
  verify_unique_nullifier(state.nullifier_set, nullifier) &&
  // Verify timing
  verify_in_registration_window(state.config, context) &&
  // Update state
  update_registration_state(state, commitment, nullifier)
}

fn update_registration_state(
  state: VotingState,
  commitment: ByteArray,
  nullifier: ByteArray,
) -> VotingState {
  // Update Merkle root with new commitment
  let new_root = update_merkle_root(state.merkle_root, commitment)
  // Add nullifier
  let new_nullifiers = add_nullifier(state.nullifier_set, nullifier)
  
  VotingState {
    ...state,
    merkle_root: new_root,
    nullifier_set: new_nullifiers,
  }
}
```

### Voting State Management
```gleam
fn verify_vote(
  state: VotingState,
  proof: ZKProof,
  commitment: ByteArray,
  nullifier: ByteArray,
  context: ScriptContext,
) -> Bool {
  // Verify phase
  verify_phase(state.phase, Voting) &&
  // Verify membership proof
  verify_membership_proof(proof, state.merkle_root) &&
  // Verify vote proof
  verify_vote_proof(proof, commitment) &&
  // Verify nullifier
  verify_unique_nullifier(state.nullifier_set, nullifier) &&
  // Verify timing
  verify_in_voting_window(state.config, context) &&
  // Update state
  update_voting_state(state, commitment, nullifier)
}

fn update_voting_state(
  state: VotingState,
  commitment: ByteArray,
  nullifier: ByteArray,
) -> VotingState {
  VotingState {
    ...state,
    vote_commitments: append(state.vote_commitments, commitment),
    nullifier_set: add_nullifier(state.nullifier_set, nullifier),
  }
}
```

### Tallying State Management
```gleam
fn verify_tally(
  state: VotingState,
  proof: ZKProof,
  result: ByteArray,
  context: ScriptContext,
) -> Bool {
  // Verify phase
  verify_phase(state.phase, Tallying) &&
  // Verify minimum participation
  verify_threshold(state.vote_commitments, state.config.min_participants) &&
  // Verify timing
  verify_tally_timing(state.config, context) &&
  // Verify result proof
  verify_tally_proof(
    proof,
    state.vote_commitments,
    result,
  ) &&
  // Update state
  update_tally_state(state, result)
}

fn update_tally_state(
  state: VotingState,
  result: ByteArray,
) -> VotingState {
  VotingState {
    ...state,
    phase: Completed,
    result: Some(result),
  }
}
```

## 3. Verification Functions

### ZK Proof Verification
```gleam
fn verify_zk_proof(
  proof: ZKProof,
  public_inputs: List<ByteArray>,
) -> Bool {
  // Interface with off-chain verification
  verify_proof(proof, public_inputs)
}

fn verify_membership_proof(
  proof: ZKProof,
  merkle_root: ByteArray,
) -> Bool {
  // Verify Merkle membership
  verify_proof(proof, [merkle_root])
}

fn verify_vote_proof(
  proof: ZKProof,
  commitment: ByteArray,
) -> Bool {
  // Verify vote validity
  verify_proof(proof, [commitment])
}
```

## 4. Helper Functions

### State Verification
```gleam
fn verify_phase(
  current: ProtocolPhase,
  expected: ProtocolPhase,
) -> Bool {
  current == expected
}

fn verify_unique_nullifier(
  set: OrderedSet<ByteArray>,
  nullifier: ByteArray,
) -> Bool {
  !contains(set, nullifier)
}

fn verify_threshold(
  votes: List<ByteArray>,
  min: Int,
) -> Bool {
  length(votes) >= min
}
```

### Timing Verification
```gleam
fn verify_in_registration_window(
  config: BallotConfig,
  context: ScriptContext,
) -> Bool {
  let current_time = get_current_time(context)
  current_time >= config.start_time &&
  current_time <= config.end_time
}

fn verify_in_voting_window(
  config: BallotConfig,
  context: ScriptContext,
) -> Bool {
  // Similar to registration window
}

fn verify_tally_timing(
  config: BallotConfig,
  context: ScriptContext,
) -> Bool {
  // Verify after voting period
}
```
