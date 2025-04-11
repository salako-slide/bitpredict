# BitPredict: Decentralized Bitcoin Prediction Markets

[![Clarity Version](https://img.shields.io/badge/Clarity-2.0-blue.svg)](https://docs.stacks.co/docs/clarity/)

A non-custodial prediction market protocol for Bitcoin price speculation, built on Stacks L2 with Clarity smart contracts.

## Overview

BitPredict enables decentralized creation and participation in BTC price prediction markets using STX tokens. Participants stake on price direction ("up"/"down") during specified time windows, with Chainlink-compatible oracle resolution and automatic payout distribution.

## Key Features

- **BTC Price Markets**: Create markets for specific price observation periods
- **STX Staking**: Minimum 1 STX stake with "up"/"down" positions
- **Oracle Resolution**: Designated address settles markets with final price
- **Auto Payouts**: Instant winnings distribution with 2% protocol fee
- **Real-Time Analytics**: On-chain market tracking and user position monitoring
- **Admin Controls**: Configurable parameters for platform governance

## Technical Specification

### Contract Architecture

#### Core Components

- **Markets Registry**: Time-bound prediction windows with price tracking
- **Staking Engine**: STX token management for positions and payouts
- **Oracle Module**: Price settlement and market resolution
- **Fee System**: Protocol revenue collection and withdrawal

### State Variables

| Variable         | Type        | Description                           |
| ---------------- | ----------- | ------------------------------------- |
| `oracle-address` | `principal` | Authorized price resolution address   |
| `minimum-stake`  | `uint`      | 1 STX minimum participation threshold |
| `fee-percentage` | `uint`      | 2% protocol fee on winnings           |
| `market-counter` | `uint`      | Sequential market ID generator        |

### Data Structures

#### Market Object

```clarity
{
  start-price: uint,    // Starting BTC price (satoshis)
  end-price: uint,     // Final oracle-reported price
  total-up-stake: uint, // Aggregate "up" positions
  total-down-stake: uint, // Aggregate "down" positions
  start-block: uint,    // Market activation block
  end-block: uint,      // Market expiration block
  resolved: bool        // Settlement status
}
```

#### User Prediction

```clarity
{
  prediction: (string-ascii 4), // "up" or "down"
  stake: uint,                  // STX amount committed
  claimed: bool                 // Payout status
}
```

## Core Functions

### Market Operations

1. **Create Market**  
   `(create-market (start-price uint) (start-block uint) (end-block uint))`

   - Admin-only market initialization
   - Requires future time window and positive starting price

2. **Place Prediction**  
   `(make-prediction (market-id uint) (prediction (string-ascii 4)) (stake uint))`

   - STX transfer to contract required
   - Validates active market window and minimum stake

3. **Resolve Market**  
   `(resolve-market (market-id uint) (end-price uint))`

   - Oracle-only execution after market expiry
   - Sets final price and marks market resolved

4. **Claim Winnings**  
   `(claim-winnings (market-id uint))`
   - Calculates pro-rata payout based on pool shares
   - Deducts protocol fee, marks position as claimed

### Administrative Functions

- `(set-oracle-address)`: Update price resolution authority
- `(set-minimum-stake)`: Adjust participation threshold
- `(set-fee-percentage)`: Modify protocol fee (≤100%)
- `(withdraw-fees)`: Extract accumulated protocol revenue

### View Functions

- `(get-market)`: Returns full market details
- `(get-user-prediction)`: Shows individual position status
- `(get-contract-balance)`: Displays total STX holdings

## Error Codes

| Code | Description                      |
| ---- | -------------------------------- |
| u100 | Unauthorized admin action        |
| u101 | Nonexistent market/user position |
| u102 | Invalid prediction type          |
| u103 | Market inactive/resolved         |
| u104 | Duplicate payout claim           |
| u105 | Insufficient STX balance         |
| u106 | Invalid parameter input          |

## Security Model

### Guarantees

- **Non-custodial Design**: Users retain STX control until market resolution
- **Immutable Rules**: Market terms enforced by smart contract
- **Clarity Safety**: Bitcoin-layer security with Stacks L2 execution

### Trust Assumptions

- Oracle integrity for price feeds
- Admin responsibility for parameter changes
- STX wallet security for participants

## Usage Example

1. **Market Creation**  
   Admin creates 100-block window market starting at BTC $50,000:

   ```clarity
   (create-market u50000000 u150000 u150100)
   ```

2. **User Participation**  
   Alice stakes 5 STX on "up" position in market #5:

   ```clarity
   (make-prediction u5 "up" u5000000)
   ```

3. **Market Resolution**  
   Oracle reports $52,000 final price at block 150100:

   ```clarity
   (resolve-market u5 u52000000)
   ```

4. **Payout Claim**  
   Alice claims winnings from successful prediction:
   ```clarity
   (claim-winnings u5) → Returns 8.16 STX (after fees)
   ```
