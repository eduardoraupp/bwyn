# Story: Withdraw money from a banking account

- **Story**: 005-as-a-user-i-want
- **Created**: 2026-10-27
- **Status**: Draft

## Story
As a user, I want to withdraw money from my banking account, so that I can access my funds.

## Acceptance tests
1. **Given** a user has a sufficient balance, **When** they request a withdrawal of a valid amount, **Then** their account balance is reduced by that amount and they receive confirmation.
2. **Given** a user has insufficient funds, **When** they request a withdrawal that exceeds their balance, **Then** the withdrawal is rejected and their balance remains unchanged.
3. **Given** a user requests a withdrawal, **When** the response is returned, **Then** it is delivered within 2 seconds under normal conditions (Driver: Quality Attributes).

## Business rules and constraints
- Withdrawal amounts must be greater than zero.
- A withdrawal must not reduce the account balance below zero (assumption: no overdraft facility exists unless stated otherwise).
- The system uses PostgreSQL as the data store for account balances (ADR 0001).
- The backend is implemented with Spring Boot (ADR 0002).

## Out of scope
- Overdraft or credit facilities.
- Integration with external payment or banking networks.
- Currency conversion.

## Follow-up stories
- Deposit money: allow a user to add funds to their account.
- View transaction history: allow a user to see a record of past withdrawals and deposits.

## Open questions
- [NEEDS CLARIFICATION: This project is described as a cinema ticketing system; a banking withdrawal does not appear to belong to it. Is this story intended for a different project, or should it be reframed as a refund or payment reversal within the ticketing context?]
- [NEEDS CLARIFICATION: Is there a maximum withdrawal limit per transaction or per day that must be enforced?]
- [NEEDS CLARIFICATION: Should a withdrawal be recorded as an auditable transaction event, or is updating the balance alone sufficient?]
