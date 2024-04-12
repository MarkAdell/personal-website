---
title: PostgreSQL Isolation Levels and Locking Summary
type: page
date: 2024-02-16
topic: sql
ShowToc: true
---

I initially prepared this summary for my myself. However, I thought other people would find it useful, so I refined it as necessary and decided to share it.

Please note that this serves only as a summary or even a cheat sheet. If you are new to these topics, it's advisable to do further reading.

## Read Phenomena

Read phenomena are potential issues (anomalies) that can occur in a database when multiple transactions are executed concurrently.

- **Dirty Read**: A transaction reads data written by another concurrent uncommitted transaction.
    - It's like reading a draft that someone else is still writing.
- **Nonrepeatable Read**: A transaction re-reads columns and finds that their values have been changed due to another concurrently committed transaction.
    - It's like re-reading a page of a book and finding that the words have changed since your first read.
- **Phantom Read**: A transaction re-reads a set of rows satisfying a specific criteria and finds newly added or removed rows due to another concurrently committed transaction.
    - It's like counting a group of people, then finding the number changes when you count again because new people have left or joined.
- **Serialization Anomaly**: The state of data resulting from executing transactions concurrently differs from the state resulting from running them one at a time.
    - It's like when you are baking a cake, it might taste differently if you added all the ingredients at the same time than if you added them one at a time in a specific order.

## Isolation Levels

Isolation levels determine how transactions influence each other.

### `Read Uncommitted`

- **Dirty Read**: Possible, but not in PostgreSQL.
- **Nonrepeatable Read**: Possible.
- **Phantom Read**: Possible.
- **Serialization Anomaly**: Possible.

`READ UNCOMMITTED` behaves the same as `READ COMMITTED` in PostgreSQL.

### `Read Committed`

- **Dirty Read**: Not possible.
- **Nonrepeatable Read**: Possible.
- **Phantom Read**: Possible.
- **Serialization Anomaly**: Possible.

`READ COMMITTED` is the default isolation level in PostgreSQL.

### `Repeatable Read`

- **Dirty Read**: Not possible.
- **Nonrepeatable Read**: Not possible.
- **Phantom Read**: Possible, but not in PostgreSQL.
- **Serialization Anomaly**: Possible.

### `Serializable`

- **Dirty Read**: Not possible.
- **Nonrepeatable Read**: Not possible.
- **Phantom Read**: Not possible.
- **Serialization Anomaly**: Not possible.

You can set the transaction isolation level in PostgreSQL using the following syntax:

```sql
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Your queries
COMMIT;
```

## Locking (Row-level locking)

Locking is a mechanism that prevents concurrent transactions from interfering with each other when accessing or modifying the same data.

One of the primary use cases for locking is to prevent anomalies such as the **Lost Update**, which can occur in lower isolation levels such as `READ COMMITTED`.

The **Lost Update** anomaly occurs when two transactions attempt to update the same data concurrently, and the changes made by one transaction are overwritten by the other, resulting in the loss of the first transaction updates.

There are two common approaches to locking: pessimistic locking and optimistic locking.

### Pessimistic Locking

Pessimistic locking assumes that conflicts will happen and so it prevents them by acquiring locks on the data. This means that a transaction will block other concurrent transactions from accessing the same data until it completes. There are two types of pessimistic locks: Shared lock and Exclusive lock.

**Shared Lock:** A shared lock allows multiple transactions to read the same data concurrently but prevents them from updating it.

**Exclusive Lock:** An exclusive lock restricts access to data to a single transaction, preventing other transactions from reading or updating it.

Pessimistic locking is often used when the likelihood of conflicts is high.

When selecting rows in postgreSQL, you can use `FOR UPDATE`, `FOR NO KEY UPDATE`, `FOR SHARE` and `FOR KEY SHARE` locking modes to explicitly lock the selected rows. Each mode provides a different level of restriction in terms of which operations other concurrent transactions are allowed to do.

Here is an example of using `SELECT FOR UPDATE`, which creates an exclusive lock.

```sql
BEGIN;
SELECT * FROM products WHERE id = 1 FOR UPDATE;
-- Other transactions trying to read or update the same row will be blocked until this transaction completes
UPDATE products SET price = 10 WHERE id = 1;
COMMIT;
```

Please note that normal `SELECT` statements under an isolation level like `READ COMMITTED` will not be blocked by a `SELECT FOR UPDATE` or any other locking mode, as normal `SELECT` statements don't acquire any locks on the rows they read, so they don't cause any conflicts with other transactions.

#### Lock modes ordered by restrictiveness from highest to lowest:

- **`SELECT .. FOR UPDATE` (exclusive lock)**
    - Blocks concurrent transactions on the same rows from performing `SELECT FOR UPDATE`, `SELECT FOR NO KEY UPDATE`, `SELECT FOR SHARE`, `SELECT FOR KEY SHARE`, `UPDATE`, and `DELETE`.
    - This is the mode implicitly used for `DELETE` commands, and `UPDATE` commands on columns with unique indexes.
    - Use when you want to prevent other transactions from modifying or acquiring any type of lock on the selected rows.
- **`SELECT .. FOR NO KEY UPDATE` (shared lock)**
    - Blocks concurrent transactions on the same rows from performing `SELECT FOR UPDATE`, `SELECT FOR NO KEY UPDATE`, `SELECT FOR SHARE`, `DELETE`, and any `UPDATE` that changes key values (such as primary keys).
    - This is the mode implicitly used for `UPDATE` commands that don't acquire a `FOR UPDATE` lock.
    - Use when you want to lock selected rows against concurrent updates that change key values and from exclusive locks, while allowing other transactions to acquire `SELECT FOR KEY SHARE` locks.
- **`SELECT .. FOR SHARE` (shared lock)**
    - Blocks concurrent transactions on the same rows from performing `SELECT FOR UPDATE`, `SELECT FOR NO KEY UPDATE`, `UPDATE`, and `DELETE`.
    - Use when you want to lock selected rows against all concurrent updates and from exclusive locks, while allowing other transactions to acquire `SELECT FOR SHARE` or `SELECT FOR KEY SHARE` locks.
- **`SELECT .. FOR KEY SHARE` (shared lock)** 
    - Blocks concurrent transactions on the same rows from performing `SELECT FOR UPDATE`, `DELETE`, and any `UPDATE` that changes key values (such as primary keys).
    - Use when you want to lock selected rows against concurrent updates that change key values and from exclusive locks, while allowing other transactions to acquire any type of shared lock.
    
#### Deadlocks

A deadlock happens when two or more transactions are waiting for each other to release locks. PostgreSQL periodically checks for deadlocks and handles them.

If a lock isn't released within the configurable parameter `deadlock_timeout` (default is 1 second), PostgreSQL will trigger the deadlock detection algorithm and if a deadlock was found, one of the involved transactions will be rolled back.

### Optimistic Locking

Optimistic locking doesn't assume that conflicts will happen, so it allows all transactions to run concurrently. If a conflict is detected at commit time (two concurrent transactions have modified the same data), one transaction will proceed and the other will be rolled back and must retry.

Optimistic locking is often used when the likelihood of conflicts is low.

In PostgreSQL, you can implement optimistic locking using a versioning field in your tables. Here's an example:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    price NUMERIC,
    version INT DEFAULT 1
);
```

When updating a row, increment the version and include it in the WHERE clause:

```sql
UPDATE 
    products 
SET 
    price = 10.0,
    version = version + 1
WHERE
    id = 1 AND version = 1
RETURNING count(id);
```

If no rows are updated, it means another transaction has already updated the row and your transaction should retry.

## Conclusion

I hope you found this summary helpful. Please let me know if you discovered any errors or have any suggestions on what to add or update, while still keeping it concise.

## Further Reading

[PostgreSQL documentation](https://www.postgresql.org/docs/current/mvcc.html)