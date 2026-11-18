# Week 46 — Transactions and Concurrency: Exercises

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

### Task 1.1: Write an Order Transaction

Write a complete transaction that processes a new order in TrailShop:

1. Insert a new order for an existing customer.
2. Insert 2–3 order items.
3. Decrease `stock_quantity` for each ordered product.
4. Include a check that stock doesn't go negative (if it would, ROLLBACK).
5. Use a SAVEPOINT before the stock update step.
> [!NOTE]
> **_Your SQL_**
>
> ```sql
> BEGIN;
>
>
>
> INSERT INTO orders (customer_id, order_date, status)
> VALUES (1, NOW(), 'pending');
>
>
> INSERT INTO order_items (order_id, product_id, quantity, unit_price)
> VALUES
>    (currval('orders_order_id_seq'), 1, 2, 89.99),
>    (currval('orders_order_id_seq'), 2, 1, 129.00),
>    (currval('orders_order_id_seq'), 3, 2, 12.50);
>
> 
> SAVEPOINT before_stock_update;
>
> UPDATE products SET stock_quantity = stock_quantity - 2 WHERE product_id = 1;
> UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 2;
> UPDATE products SET stock_quantity = stock_quantity - 2 WHERE product_id = 3;
>
> -- 4. stock check, if something went under 0 stop everything
> DO $$
> BEGIN
>    IF EXISTS (SELECT 1 FROM products
>               WHERE product_id IN (1, 2, 3)
>                 AND stock_quantity < 0) THEN
>        RAISE EXCEPTION 'Not enough stock, order cancelled';
>    END IF;
> END $$;
>
> COMMIT;
>
>
> ```

Test the transaction by:

- Running it successfully with sufficient stock.
- Running it where stock is insufficient — verify the ROLLBACK works.

### Task 1.2: Observe READ COMMITTED Behavior (Optional)

> [!TIP]
> **Recommended practice.** Do Tasks 1.2–1.4. They are not required to finish the TrailShop project. They prepare you for the exams.

Open two psql sessions (or two query windows). Demonstrate a non-repeatable read:

1. Session A: `BEGIN;` then `SELECT stock_quantity FROM products WHERE product_id = 101;`
2. Session B: `BEGIN;` then `UPDATE products SET stock_quantity = stock_quantity - 5 WHERE product_id = 101;` then `COMMIT;`
3. Session A: `SELECT stock_quantity FROM products WHERE product_id = 101;` (observe different value)
4. Session A: `COMMIT;`

Document what you observe and explain why it happens at READ COMMITTED.

### Task 1.3: Observe Locking (Optional)

Open two sessions and demonstrate row-level locking:

1. Session A: `BEGIN;` then `SELECT * FROM products WHERE product_id = 101 FOR UPDATE;`
2. Session B: Try `UPDATE products SET price = 99.99 WHERE product_id = 101;` — observe it waits.
3. Session A: `COMMIT;` — observe Session B proceeds.

Document the behavior.

### Task 1.4: Experiment with Isolation Levels (Optional)

1. Repeat Task 1.2, but set Session A to `REPEATABLE READ`. Does the non-repeatable read still occur?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_
>
> 2. In a REPEATABLE READ transaction, try to UPDATE a row that another transaction has already committed a change to. What error do you get?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_
>
> 3. Document the differences between READ COMMITTED and REPEATABLE READ behavior.

---

## 2. Theory Review Questions

Answer in your own words (2–4 sentences each):

1. What is a transaction, and why is it important for data integrity?

> [!NOTE]
> **_Your Answer_**
>
> _(A transaction is a group of SQL statements that are run as one unit, either all of them work or none of them.)_

2. Explain the Atomicity property using a real-world example (not from the lecture notes).

> [!NOTE]
> **_Your Answer_**
>
> _(When i transfer money from my bank account to a friend, the bank takes money from my account and adds it to his. If the system crashes after taking my money but before adding it to his, atomicity makes sure the whole transfer is cancelled, so the money doesnt just disappear.)_
>
> 3. What does Isolation guarantee? Why is it needed in a multi-user system?

> [!NOTE]
> **_Your Answer_**
>
> _(Isolation means that transactions running at the same time dont mess with each other, each one acts like it is alone in the database. Its needed because in a shop many users buy, update stock and read data at the same moment and without it they could see half finished changes or overwrite each others work.)_

4. What is the difference between COMMIT and ROLLBACK?

> [!NOTE]
> **_Your Answer_**
>
> _("COMMIT" saves all the changes from the transaction permanently and other users can see them. "ROLLBACK" cancels all the changes made in the transaction like it never happened..)_

5. Explain what a SAVEPOINT is and give a scenario where it's useful.

> [!NOTE]
> **_Your Answer_**
>
> _(A "SAVEPOINT" is a checkpoint inside a transaction, you can go back to it with "ROLLBACK TO SAVEPOINT" and only the work after it is undone, the transaction stays open. For example when making an order with many items, if one item is out of stock i can roll back only that item and still finish the rest of the order.)_
>
> 6. What is a dirty read? Why does PostgreSQL never allow it?

> [!NOTE]
> **_Your Answer_**
>
> _(A dirty read is when a transaction reads data that another transaction changed but did not commit yet, and if that one rolls back you read something that never really existed. PostgreSQL never allows it because of MVCC, every transaction only sees committed versions of rows.)_

7. Describe the lost update problem. How can SELECT FOR UPDATE prevent it?

> [!NOTE]
> **_Your Answer_**
>
> _(Lost update is when two transactions read the same value, both change it and save, and the second one overwrites the first ones change. "SELECT FOR UPDATE" locks the row when you read it, so the second transaction has to wait until the first one commits and then it reads the new correct value. safe guard basically)_

8. What is the default isolation level in PostgreSQL? What concurrency problems does it still allow?

> [!NOTE]
> **_Your Answer_**
>
> _(The default is "READ COMMITTED". Dirty reads cant happen, but non-repeatable reads and phantom reads can still happen, also lost updates if you dont lock rows.)_

9. Explain MVCC in one paragraph: what is it, and what advantage does it provide?

> [!NOTE]
> **_Your Answer_**
>
> _(Multi-Version Concurrency Control means PostgreSQL keeps several versions of a row instead of overwriting it right away. When a row is updated a new version is made and the old one stays for transactions that still need it, so each transaction sees a snapshot of committed data.)_

10. What is a deadlock, and how does PostgreSQL handle it when one is detected?

> [!NOTE]
> **_Your Answer_**
>
> _(A deadlock is when two transactions wait for each other, T1 has a lock that T2 needs and T2 has a lock that T1 needs, so neither can go on. PostgreSQL detects it automatically, aborts one of the transactions with an error and lets the other one continue, then the app has to retry the aborted one. ((or it could mean valves game :D)))_

---

## 3. Scenario Analysis (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each scenario, identify: (a) which concurrency problem occurs, and (b) at which isolation level(s) it can happen.

### Scenario A

Transaction 1 reads customer 5's order count (3 orders). Transaction 2 inserts a new order for customer 5 and commits. Transaction 1 reads customer 5's order count again and gets 4.

### Scenario B

Transaction 1 reads product 101's price (€89.99). Transaction 2 updates product 101's price to €79.99 but then rolls back. If Transaction 1 had somehow seen €79.99, what problem would that be?

### Scenario C

Two warehouse workers simultaneously check stock for product 101 (both see 10). Worker A ships 3 units and sets stock to 7. Worker B ships 4 units and sets stock to 6. The correct stock should be 3.

---

## 4. Transaction Writing (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write complete SQL transactions for the following TrailShop scenarios. Include BEGIN, appropriate operations, error handling (SAVEPOINT where useful), and COMMIT.

### Transaction 1: Bulk Price Update

Update the price of all products in the "Footwear" category by +10%. If any product's new price would exceed €200, rollback the entire update.

### Transaction 2: Customer Account Deletion

Delete a customer and all their associated data:

1. Delete their order_items (for all their orders)

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_
>
> 2. Delete their orders

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_
>
> 3. Delete the customer record

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

Use SAVEPOINTs between each step in case of errors.

### Transaction 3: Inventory Restock

Restock 3 products (add specific quantities to stock). Use SELECT FOR UPDATE to lock the rows first, preventing lost updates. Include a check that the new stock doesn't exceed a maximum (e.g., 200).

---

## 5. True/False

For each statement, write TRUE or FALSE and briefly explain why.

1. In PostgreSQL, a single SQL statement outside of an explicit BEGIN/COMMIT is not a transaction.
_(FALSE – PostgreSQL uses autocommit so every single statement is its own transaction and commits automatically.)_
2. ROLLBACK undoes all changes made since the last SAVEPOINT.
_(FALSE – plain ROLLBACK undoes the whole transaction.)_
3. At READ COMMITTED isolation level, a transaction can never see another transaction's uncommitted changes.
_(TRUE – READ COMMITTED only shows committed data, so dirty reads cant happen.)_
4. REPEATABLE READ in PostgreSQL prevents phantom reads.
_(TRUE – the SQL standard allows phantoms at this level, but PostgreSQL uses one snapshot for the whole transaction.)_
5. SELECT FOR UPDATE blocks other SELECT statements on the same rows.
_(FALSE – normal SELECTs still work because of MVCC.)_
6. If a deadlock occurs, PostgreSQL will wait indefinitely for it to resolve.
_(FALSE – PostgreSQL detects the deadlock and aborts one of the transactions so the other can continue.)_
7. MVCC means PostgreSQL keeps multiple versions of each row, allowing readers and writers to work concurrently without blocking.
_(TRUE – each transaction reads its own snapshot of committed versions, so reading and writing dont block each other.)_
8. Once a transaction is committed, its changes can be undone with ROLLBACK.
_(FALSE – after COMMIT the changes are permanent.)_
---

## Submission Checklist

**Required**

- [ ] Order transaction written and tested, including the insufficient-stock rollback (Task 1.1)
- [ ] Theory review questions answered
- [ ] True/False questions answered with a short explanation

**Recommended practice**

- [ ] READ COMMITTED, locking, and isolation-level experiments documented (Tasks 1.2–1.4)
- [ ] Scenario analysis completed
- [ ] Extra transactions written