# StakeNets - Decentralized Consensus Protocol

## Overview

StakeNets is a Clarity smart contract that implements a stake-weighted consensus mechanism for validators to submit observations and reach agreement on values through a trust-based evaluation system. The protocol combines stake amounts with accuracy scores to determine validator influence in the consensus process.

## Key Features

- **Stake-Weighted Consensus**: Validator influence is calculated based on both stake amount and historical accuracy
- **Round-Based Submissions**: Organized voting rounds with participation tracking
- **Reputation System**: Validators build reputation through accurate submissions
- **Cooldown Periods**: Prevents spam by enforcing time delays between participations
- **Administrative Controls**: Manager can suspend validators and adjust system parameters

## Core Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `base-deposit` | 1,000,000 | Minimum stake required to register as validator |
| `confidence-threshold` | 80 | Minimum confidence level for consensus |
| `max-value` | 10,000,000 | Maximum allowed observation value |
| `volatility-cap` | 500 | 5% variation limit |
| `incentive-amount` | 100 | Reward for valid inputs |
| `fine-amount` | 50 | Penalty for invalid entries |
| `cooldown-period` | 144 blocks | ~24 hours between participations |

## Data Structures

### Validator Registry
Stores validator information:
- `stake`: Amount staked by validator
- `is-operational`: Active status
- `accuracy`: Historical accuracy score (0-100)
- `entry-count`: Total submissions made
- `privileges`: Access level
- `reputation-score`: Accumulated reputation
- `last-activity-block`: Last participation block height

### Round Data
Tracks each consensus round:
- `consensus-value`: Final agreed value (optional)
- `participation-count`: Number of validators who submitted
- `is-sealed`: Whether round is closed for new submissions
- `is-completed`: Whether consensus has been reached
- `aggregate-stake`: Total stake weight in the round

### Submission Records
Individual validator submissions per round:
- `submitted-value`: The value submitted by validator
- `stake-weight`: Calculated influence for this submission
- `is-verified`: Verification status
- `is-processed`: Processing status

## Public Functions

### Validator Operations

#### `register-validator`
Registers a new validator in the system.

**Requirements:**
- Caller must have balance ≥ `base-deposit`
- Caller must not be already registered

**Initial Values:**
- Accuracy: 90
- Privileges: 1
- Operational: true

**Returns:** `(ok true)` or error code

#### `update-stake`
Updates the stake amount for an existing validator.

**Parameters:**
- `new-stake` (uint): New stake amount

**Requirements:**
- Caller must be registered validator
- New stake ≥ `base-deposit`

**Returns:** `(ok true)` or error code

#### `record-observation`
Submits an observation value for the current round.

**Parameters:**
- `observation-value` (uint): The value to submit

**Requirements:**
- Validator must be operational
- Must pass cooldown period since last participation
- Value must be between 1 and `max-value`
- Cannot submit twice in same round

**Returns:** `(ok true)` or error code

**Effects:**
- Records submission with calculated stake weight
- Updates round participation count
- Updates validator entry count and last activity

### Round Management

#### `finalize-round`
Closes the current round and calculates consensus value.

**Requirements:**
- Round must not already be completed
- Minimum 3 validators must have participated

**Effects:**
- Seals the round
- Calculates median consensus value
- Initiates new round

**Returns:** `(ok consensus-value)` or error code

### Administrative Functions

#### `suspend-validator`
Suspends a validator from participating (Manager only).

**Parameters:**
- `target` (principal): Address of validator to suspend

**Requirements:**
- Caller must be manager
- Target must be operational
- Cannot suspend the manager

**Returns:** `(ok true)` or error code

#### `adjust-parameters`
Updates system parameters (Manager only).

**Parameters:**
- `new-cooldown` (uint): New cooldown period in blocks

**Requirements:**
- Caller must be manager
- Cooldown must be between 1 and 1000 blocks

**Returns:** `(ok true)` or error code

## Read-Only Functions

### `view-round`
Retrieves information about a specific round.

**Parameters:**
- `round` (uint): Round number

**Returns:** Round data or none

### `view-submission`
Retrieves a validator's submission for a specific round.

**Parameters:**
- `round` (uint): Round number
- `validator` (principal): Validator address

**Returns:** Submission data or none

## Error Codes

| Code | Description |
|------|-------------|
| u1 | Validator not registered |
| u2 | Validator not operational |
| u3 | Cannot participate (cooldown or duplicate) |
| u4 | Round data not found |
| u5 | Round already completed |
| u6 | Insufficient participation (< 3 validators) |
| u7 | Invalid input value |
| u8 | Insufficient stake for registration |
| u9 | Validator already registered |
| u10 | New stake below minimum |
| u11 | Unauthorized (not manager) |
| u12 | Invalid suspension target or parameter |
| u13 | Validator not found |

## Workflow Example

1. **Registration:**
   ```clarity
   (contract-call? .stakenets register-validator)
   ```

2. **Submit Observation:**
   ```clarity
   (contract-call? .stakenets record-observation u5000000)
   ```

3. **Finalize Round (after ≥3 submissions):**
   ```clarity
   (contract-call? .stakenets finalize-round)
   ```

4. **View Results:**
   ```clarity
   (contract-call? .stakenets view-round u0)
   ```

## Influence Calculation

Validator influence is calculated using:
```
stake-factor = (stake / 1,000,000)
accuracy-factor = (accuracy / 100)
influence = stake-factor × accuracy-factor
```

This ensures validators with higher stakes and better accuracy have proportionally more weight in consensus.

## Security Considerations

- **Manager Privileges**: The deploying address has elevated permissions
- **Cooldown Protection**: Prevents rapid-fire submissions
- **Input Validation**: All values checked against maximum limits
- **Suspension Protection**: Manager cannot suspend themselves

## Current Limitations

- Median calculation uses placeholder logic (requires production implementation)
- Balance checking is simplified (production needs actual balance queries)
- No slashing mechanism for malicious behavior
- No reward distribution implementation

## Future Enhancements

- Implement actual stake-weighted median calculation
- Add reward/penalty distribution
- Implement validator slashing for malicious behavior
- Add dispute resolution mechanism
- Implement time-weighted accuracy tracking
