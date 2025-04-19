# Concurrency

### Exclusive vs Shared Locks

- **Exclusive Lock**: When acquired, if readers try to access the row it throws the error
- **Shared Lock**: When acquired, if writers try to access the row it throws the error
- To acquire a shared lock, the row must not be locked by an exclusive lock
- To acquire an exclusive lock, the row must not be locked by any other lock

### Two Phase Lock

- **Growing Phase**: Locks are acquired but not released
- **Shrinking Phase**: Locks are released but not acquired
- **Deadlock**: A situation where two or more transactions are waiting for each other to release locks, causing a standstill

Example of 2-phase locking:

```sql
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SELECT * FROM accounts WHERE id = 2 FOR UPDATE;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```
- In this example, the transaction acquires locks on the rows for both accounts before updating them. This ensures that no other transaction can modify these rows until the current transaction is complete.
- This is an example of 2-phase locking because the transaction first acquires locks (growing phase) and then releases them (shrinking phase) when it commits.
- If another transaction tries to acquire a lock on one of these rows while the first transaction is still in the growing phase, it will be blocked until the first transaction commits or rolls back.
- If the first transaction tries to acquire a lock on a row that is already locked by another transaction, it will be blocked until the other transaction releases the lock.
- This can lead to deadlocks if two transactions are waiting for each other to release locks. In such cases, the database management system will typically detect the deadlock and abort one of the transactions to resolve the situation.
- **Lock Granularity**: The size of the lock, which can be at the database, table, or row level
- **Lock Contention**: When multiple transactions are trying to acquire locks on the same resource, leading to delays and potential deadlocks
- **Lock Escalation**: The process of converting many fine-grained locks into a single coarse-grained lock to reduce overhead