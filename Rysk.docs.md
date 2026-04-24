# Rysk-like DeFi App Research and Requirements

## Overview
This document outlines the requirements for a DeFi application inspired by Rysk Finance. The product is centered on five strategies and a centralized Market Maker (MM) that handles pricing, execution, custody, and withdrawals.

## Core Requirements

### Strategy Set
The application should support five distinct financial strategies:
1. Covered Call
2. Cash-Secured Put
3. Iron Condor
4. Yield Maximizer
5. Managed Volatility

### Market Maker Integration
The system should use a centralized architecture with a dedicated MM.
- The MM provides RFQ pricing through an API.
- The user selects a strategy and submits a form in the frontend.
- The backend sends the request to the MM API.
- The MM returns quote data such as yield, APY, strike, and other strategy parameters.

### Custody and Wallets
The custody model should be built around the MM’s Fireblocks setup.
- User funds are deposited to the MM’s Fireblocks omnibus account.
- The system uses a single Fireblocks address for pooled user deposits.
- A social login wallet should be used for user onboarding and wallet creation.
- Suitable providers include Web3Auth, Magic, Dynamic, or a similar MPC-based solution.

### Lifecycle Management
The application should support strategy expiration and withdrawals.
- When a position expires, the balance becomes available for new positions.
- Withdrawal flow:
  1. The user submits a withdrawal request to the MM.
  2. The MM reviews and approves the request.
  3. The user claims the funds.

## Research Topics

### Rysk Finance Architecture
Research has focused on Rysk V1 and V1.2.
- Physical settlement is used.
- Positions are 100% collateralized.
- The RFQ engine supports options such as calls and puts.

### Fireblocks API and SDK
The preferred custody model for this type of retail application is the omnibus account model.
- A single vault is used to pool assets.
- Internal ledgers track user balances.
- This model is operationally efficient for MM-centric custody.

### Social Login Providers
Web3Auth and Privy were identified as strong options for mapping social identities to wallet and vault identifiers.

### RFQ System
The key implementation pattern is:
1. Frontend form
2. Backend processing
3. MM API request
4. Frontend quote display

### Strategy Definitions
1. Covered Call (ETH/BTC)
   - Hold the underlying asset and sell upside via call options.
   - The MM provides the bid for the call.
2. Cash-Secured Put (USDC)
   - Hold stablecoins and earn yield by committing to buy at a lower price.
   - The MM provides the bid for the put.
3. Iron Condor
   - A neutral strategy using both calls and puts.
   - Best suited for sideways markets.
4. Yield Maximizer (Lending + Options)
   - Principal stays in Aave.
   - Yield is used to buy out-of-the-money options.
5. Managed Volatility (DHV)
   - User funds are pooled.
   - The MM dynamically hedges to target a specific delta/gamma profile.
   - This reproduces the core Rysk product.

## Proposed Tech Stack

| Component | Recommendation | Why |
| --- | --- | --- |
| Auth and Wallet | Dynamic | Native Fireblocks integration and strong social onboarding |
| Custody | Fireblocks Omnibus | High security, gas efficient, and MM-friendly |
| Frontend | Next.js + Tailwind | Fast development, SEO-friendly, and modern UI support |
| Backend | NestJS (Node.js) | Strong typing, modular structure, and good API integration support |
| Database | PostgreSQL | Reliable source of truth for the internal ledger |
| Blockchain | Arbitrum / Base | Low fees and strong liquidity for WETH and USDC |

## Implementation Roadmap

### Phase 1: Integration and Onboarding
- Set up Dynamic with Fireblocks NCW.
- Integrate the MM API for basic deposit address generation.
- Build the internal ledger database to track user balances.

### Phase 2: RFQ and Execution
- Implement the RFQ frontend form.
- Connect to the MM API for live quotes.
- Build the accept and lock flow from backend to MM.

### Phase 3: Lifecycle and Expiration
- Automate expiration monitoring.
- Implement a roll-to-new-position feature using internal balance.

### Phase 4: Withdrawals
- Implement the withdrawal request flow.
- Build the claim mechanism that triggers the MM Fireblocks outbox.

## System Architecture

### Workflow Overview
The system can be described as four main stages:

1. Onboarding and deposit
   - The user logs in through a social wallet provider.
   - The wallet is created through Dynamic or a similar MPC onboarding flow.
   - The user transfers funds to the MM omnibus address.
   - Fireblocks confirms the deposit.
   - The backend updates the internal ledger.

2. RFQ and execution
   - The user selects a strategy and enters a size.
   - The backend requests a live quote from the MM API.
   - The MM returns pricing and strategy terms.
   - The frontend shows the quote with a short timer.
   - If the user accepts, the backend locks the balance and sends the execution command.

3. Expiration and balance update
   - The MM notifies the backend when a position expires.
   - The backend unlocks principal plus yield in the internal ledger.
   - The app updates the user’s available balance.

4. Withdrawal and claim
   - The user submits a withdrawal request.
   - The MM reviews and approves it.
   - Funds are moved into claimable escrow.
   - The app marks the withdrawal as claimable.
   - The user claims the funds through the app.

### Workflow Diagram
![Workflow Diagram](Rysk.workflow.png)

### Detailed Architecture Notes

#### 1. User Authentication and Wallet
- Recommended providers are Dynamic, Web3Auth, or Privy.
- A non-custodial MPC wallet is created during social login.
- The social wallet is the source of funds and the destination for claims.
- It is separate from the MM’s Fireblocks omnibus vault.

#### 2. RFQ Engine
- The user selects a strategy such as an ETH covered call and enters an amount.
- The backend verifies the user’s internal balance.
- The backend calls the MM API with the quote request.
- The MM returns pricing data such as APY, yield, and strike price.
- The frontend displays the quote with a timer.
- If the user accepts, the backend locks the balance and sends the execution command.

#### 3. Custody and Settlement
- Funds are held in the MM’s Fireblocks omnibus vault.
- The app shows a virtual balance that represents the user’s share of the pooled assets.
- Deposits are attributed to the user in the internal ledger.

#### 4. Strategy Lifecycle and Expiration
- When a position is opened, the RFQ acceptance triggers a lock on the strategy funds.
- At expiration, the MM calculates the final value, including principal, yield, and fees.
- The backend updates the available balance.
- The user can roll the balance into a new position without on-chain movement.

#### 5. Withdrawal and Claim Flow
1. The user initiates a withdrawal request in the app.
2. The MM API receives the request and performs risk checks.
3. The MM approves the withdrawal.
4. Funds are moved from the MM omnibus vault to an escrow wallet or to a claim smart contract.
5. The user is notified that the funds are claimable.
6. The user submits a claim.
7. If using a claim smart contract, the backend provides a signed voucher.
8. If using escrow, the backend triggers a Fireblocks transfer to the user wallet.

## Recommendation
The safest and most transparent settlement model is a claim smart contract. It gives the user on-chain proof that funds are available and allows programmatic claiming rather than relying entirely on backend-controlled transfers.

