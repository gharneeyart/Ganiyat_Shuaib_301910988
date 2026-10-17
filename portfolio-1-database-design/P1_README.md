# Portfolio 1: Database Design and Modelling

## Case Study

A digital banking and fintech platform manages customer accounts, internal fund transfers, and transaction histories. Customers open accounts, deposit funds, and transfer money between accounts. Bank administrators manage customer profiles, monitor account statuses, and oversee platform transactions.

## Users

- Customers manage their personal profiles, open accounts, and execute fund transfers.
- Bank administrators register customer profiles, update account statuses, and review system transaction logs.

## Main Operations

- Register a customer
- Open a bank account
- Deposit funds
- Execute a fund transfer
- Withdraw funds
- View account balances
- Review transaction history

## Business Rules

- Each customer has a unique customer ID and a verified email address.
- Each bank account has a unique account number and belongs to exactly one customer. A customer can hold multiple accounts.
- Each transaction refers to one source account and one destination account. Internal transfers require both accounts to be active.
- An account balance cannot fall below zero. Transactions exceeding the available balance are rejected.
- The transaction date must be current or earlier and cannot precede the account opening date.
- Each account has a status of active, suspended, or closed. Suspended or closed accounts cannot initiate transactions.
- A deposit increases the target account balance, while a withdrawal or transfer decreases the source balance.
- Transaction records are permanent and cannot be deleted once committed.

## Scope

The scope covers a single fintech platform supporting basic customer accounts, deposits, withdrawals, and internal transfers using fictional data. Loans, credit cards, currency exchange, investment portfolios, and external payment gateways are outside this example.