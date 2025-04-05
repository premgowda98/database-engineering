# ACID

## Atomicity

1. All queries in a transaction must succeed
2. Lack of atomicity induces inconsistency

## Isolation

| pid  |  qnt  | price |
| :--- | :---: | ----: |
| P1   |  10   |    20 |
| P2   |  15   |    10 |

1. Read Phenomenon
    1. Dirty Read
        1. A transaction can see the changes by the other transactions which are not committed yet
        2. If the second transaction is rolled back, the data with the first transaction is wrong
        3. Example
           1. T1 -> Select qnt, price FROM products
           2. T2 -> Update products Set qnt = 15 Where pid = P1
           3. T1 -> Select sum(qnt*price) where pid = P1 => 300
           4. T2 -> Rollback
        4. In this case the sum obtained in the T1 is wrong since T2 has been rolled back
    2. Non Repeatable Reads
       1. Example
           1. T1 -> Select qnt, price FROM products
           2. T2 -> Update products Set qnt = 15 Where pid = P1
           3. T2 -> Commit
           4. T1 -> Select sum(qnt*price) where pid = P1 => 300
    3. Phantom Reads
       1. Example
           1. T1 -> Select qnt, price FROM products
           2. T2 -> Insert into products Values (P3, 25, 10)
           3. T2 -> Commit
           4. T1 -> Select sum(qnt*price) => 550
    4. Lost Updates
2. Isolation Levels
   1. To fix the read phenomenons
   2. Read Uncommitted
      1. No Isolation
      2. Only in SQL Server
      3. Can have dirty read
   3. Read Committed
      1. See only committed changes of other transactions
      2. Can have Non-Repeatable and Phantom Reads
   4. Repeatable Read
      1. To Non-repeatable read phenomenon
      2. The transaction will make sure that when the query reads the row, that row will remain unchanged until the transaction is completed
      3. Will not tackle phantom reads
   5. Snapshot
      1. Will cover all the read phenomenon
   6. Note: In postgres a Repeatable Read Isolation is actually a Snapshot level
   7. Serializable

## Consistency

1. Consistency in data, reads

## Durability

1. Changes made by the committed transactions must be stored in durable non-volatile storage.
2. Techniques
   1. WAL - Write Ahead Logs
   2. Asynchronous Snapshot
   3. AOF

## Demo

```sql
create table products (pid serial primary key, name text, price float, inventory integer);
create table sales (sid serial primary key, pid integer, price float, quantity integer);

insert into products (name, price, inventory) values ('p1', 25.5, 200);
```

### Atomicity

```sql
select * from products;
update products set inventory = 100;
// crash the databse

select * from products; => inventory must be 200
```

### Non-repeatable read

| Transaction 1           |                        Transaction 2 |
| :---------------------- | -----------------------------------: |
| begin transaction;      |                                      |
| select * from products; |                                      |
| includes 1 product      |                                      |
|                         |                   begin transaction; |
|                         | insert into products ('p2', 23, 500) |
|                         |                              commit; |
| select * from products; |                                      |
| includes 2 products     |                                      |

With Repeatable read isolation level

| Transaction 1                                      |                        Transaction 2 |
| :------------------------------------------------- | -----------------------------------: |
| begin transaction isolation level repeatable read; |                                      |
| select * from products;                            |                                      |
| includes 2 product                                 |                                      |
|                                                    |                   begin transaction; |
|                                                    | insert into products ('p2', 23, 500) |
|                                                    |                              commit; |
| select * from products;                            |                                      |
| includes 2 products;                               |                                      |
| commit;                                            |                                      |
| select * from products;                            |                                      |
| includes 3 products;                               |                                      |

