# Digital Banking and Fintech Platform Database Design

## 1. Entities

I chose four entities:

| Entity | Information stored |
| --- | --- |
| customer | A person who holds accounts on the platform |
| administrator | A bank staff member who manages profiles and account statuses |
| account | One bank account owned by one customer |
| bank_transaction | One committed money movement: a deposit, withdrawal or transfer |

I kept deposits, withdrawals and transfers in one table because they share the same details. A type field tells them apart.

## 2. Attributes and keys

**Unique** means no repeated values. **Required** means the value cannot be `NULL` (missing).

| Entity | Attributes and proposed PostgreSQL types | Keys and required values |
| --- | --- | --- |
| customer | customer_id BIGINT; full_name VARCHAR(120); email VARCHAR(254); email_verified_at TIMESTAMPTZ; created_at TIMESTAMPTZ | PK: customer_id. Unique: email. All required except email_verified_at. |
| administrator | admin_id BIGINT; staff_number VARCHAR(30); full_name VARCHAR(120); email VARCHAR(254) | PK: admin_id. Unique: staff_number and email separately. All required. |
| account | account_id BIGINT; account_number VARCHAR(10); customer_id BIGINT; status VARCHAR(10); balance NUMERIC(14,2); opened_at TIMESTAMPTZ; status_updated_by BIGINT | PK: account_id. FKs: customer_id → customer.customer_id; status_updated_by → administrator.admin_id. Unique: account_number. All required except status_updated_by. |
| bank_transaction | transaction_id BIGINT; transaction_type VARCHAR(10); source_account_id BIGINT; destination_account_id BIGINT; amount NUMERIC(14,2); created_at TIMESTAMPTZ | PK: transaction_id. FKs: source_account_id and destination_account_id → account.account_id. Required: transaction_type, amount, created_at. |

`NUMERIC(14,2)` stores exact money values with two decimal places, avoiding rounding errors. `TIMESTAMPTZ` stores a date and time with its time zone.

## 3. Relationships

| Tables linked | Relationship | Meaning |
| --- | --- | --- |
| customer → account | One to many | A customer can have zero or more accounts; each account belongs to exactly one customer. |
| account → bank_transaction (as source) | One to many | An account can send zero or more transactions; each withdrawal or transfer has exactly one source account. |
| account → bank_transaction (as destination) | One to many | An account can receive zero or more transactions; each deposit or transfer has exactly one destination account. |
| administrator → account | One to many | An administrator can update zero or more account statuses; each account records at most one latest updater. |

No linking table is needed because no many-to-many relationship exists.

## 4. Business rules and constraints

- **Rule 1:** email is unique and required. Account opening checks that email_verified_at is filled in. This needs an operation check, not a key.
- **Rule 2:** account_number is unique. A required customer_id FK links each account to one customer. customer_id can repeat, so one customer can hold many accounts.
- **Rule 3 (change flagged):** a deposit has no source account and a withdrawal has no destination account. A CHECK matches the type to its accounts: transfers need both, deposits need only a destination, withdrawals need only a source. Another CHECK stops an account transferring to itself. Checking that both accounts are active needs transaction logic.
- **Rule 4:** a CHECK requires balance ≥ 0, and another requires amount > 0. A withdrawal or transfer that would make the balance negative fails, and the whole transaction is rejected.
- **Rule 5:** created_at defaults to the current time. Comparing it with opened_at in another table needs a trigger (code that runs automatically on insert).
- **Rule 6:** a CHECK limits status to active, suspended or closed. Blocking transactions from non-active accounts needs transaction logic.
- **Rule 7:** each money movement inserts one bank_transaction row and updates the balances in one database transaction, so all changes succeed or none do.
- **Rule 8:** the application role has no DELETE or UPDATE permission on bank_transaction. FKs use ON DELETE RESTRICT, so accounts with history cannot be removed.

For example, a transfer of ₦5,000 from account A to account B inserts one transfer row and updates both balances together. If A holds only ₦3,000, the balance check fails and nothing is saved.

## 5. Design assumptions

1. Balance is stored, not calculated, so rule 4 is checked instantly by a constraint. It always equals the sum of the account's transactions.
2. Only the latest status change is recorded; full status history is outside scope.
3. All amounts use one currency (naira).