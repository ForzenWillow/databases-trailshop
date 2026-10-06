# Week 40 — Exercises: SQL Fundamentals

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task

This week you'll build the TrailShop database from scratch and practice manipulating data.

### Task 1.1: Create the Database

1. Open your PostgreSQL terminal (psql) or pgAdmin
2. Create a new database called `trailshop`
3. Connect to it

### Task 1.2: Create All Tables

Write and execute the CREATE TABLE statements for all five TrailShop tables in the correct order:
- categories
- customers
- products
- orders
- order_items

**Requirements:**
- Use appropriate data types for each column
- Include all constraints from the theory (NOT NULL, UNIQUE, CHECK, FOREIGN KEY, DEFAULT)
- Use SERIAL for primary keys
- Ensure foreign keys reference the correct parent tables

**Verify** by running `\dt` in psql to list all tables.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> CREATE TABLE IF NOT EXISTS categories (
>    category_id SERIAL PRIMARY KEY,
>    name VARCHAR(100) NOT NULL
>);
>
>CREATE TABLE IF NOT EXISTS customers (
>    customer_id SERIAL PRIMARY KEY,
>    email VARCHAR(255) UNIQUE,
>    name VARCHAR(100) NOT NULL
>);
>
>CREATE TABLE IF NOT EXISTS products (
>    product_id SERIAL PRIMARY KEY,
>    name VARCHAR(200) NOT NULL
>);
>
>CREATE TABLE IF NOT EXISTS product_categories (
>    product_id INTEGER REFERENCES products (product_id),
>    category_id INTEGER REFERENCES categories (category_id),
>    PRIMARY KEY (product_id, category_id)
>);
>
>CREATE TABLE IF NOT EXISTS orders (
>    order_id SERIAL PRIMARY KEY,
>    customer_id INTEGER NOT NULL REFERENCES customers(customer_id)
>);
>
>CREATE TABLE IF NOT EXISTS >order_items (
>    order_id INTEGER REFERENCES >orders(order_id),
>    product_id INTEGER REFERENCES ?products(product_id)
>);
>
>
> ```


### Task 1.3: Insert Sample Data

Insert the following data:

**Categories** (at least 5):
- Footwear, Backpacks, Tents, Clothing, Accessories

> [!NOTE]
> ***Your SQL***
>
> ```sql
> INSERT INTO categories (name)
> VALUES ('Footwear'),
>       ('Backpacks'),
>       ('Tents'),
>       ('Clothing'),
>       ('Accessories');
>
>
> ```

**Customers** (at least 5):
- Use easy to write names with realistic email addresses

> [!NOTE]
> ***Your SQL***
>
> ```sql
> INSERT INTO customers (name, email)
> VALUES ('William Earl','william.earl@gmail.com'),
>       ('Mark Jones', 'mark.jones@example.com'),
>       ('Laura Brown', 'laura.brown@example.com'),
>       ('Peter Miller', 'peter.miller@example.com'),
>       ('Emma Wilson', 'emma.wilson@example.com');
>
>
> ```

**Products** (at least 10):
- At least 2 products per category
- Prices ranging from €20 to €500
- Various stock levels

> [!NOTE]
> ***Your SQL***
>
> ```sql
> INSERT INTO product_categories (product_id, category_id)
>SELECT p.product_id, c.category_id
>FROM products p, categories c
>WHERE p.name = 'Merino Base Layer' AND c.name = 'Accessories';
>
>INSERT INTO products (name, price, stock)
>VALUES ('Trail Running Shoes', 89.90, 25),
>       ('Hiking Boots', 149.00, 12),
>       ('Daypack 30L', 59.90, 40),
>       ('Trekking Backpack 65L', 189.00, 8),
>       ('2-Person Tent', 249.00, 6),
>       ('4-Person Family Tent', 499.00, 2),
>       ('Waterproof Jacket', 129.00, 18),
>       ('Merino Base Layer', 74.50, 30),
>       ('Headlamp', 24.90, 55),
>       ('Trekking Poles', 39.90, 0);
> ```

**Orders** (at least 5):
- Different customers, different statuses

> [!NOTE]
> ***Your SQL***
>
> ```sql
> INSERT INTO orders (customer_id, order_date, status)
>SELECT c.customer_id, v.order_date::timestamp, v.status
>FROM (VALUES ('anna.smith@example.com',   '2026-09-18 10:15', 'delivered'),
>             ('mark.jones@example.com',   '2026-09-25 14:30', 'shipped'),
>             ('laura.brown@example.com',  '2026-09-30 09:05', 'paid'),
>             ('peter.miller@example.com', '2026-10-02 16:45', 'pending'),
>             ('emma.wilson@example.com',  '2026-10-03 11:20', 'cancelled'))
>     AS v(email, order_date, status)
>JOIN customers c ON c.email = v.email;
>
>
> ```

**Order Items** (at least 10):
- Multiple items in some orders, single items in others

> [!NOTE]
> ***Your SQL***
>
> ```sql
>   INSERT INTO order_items (order_id, product_id, quantity, unit_price)
>SELECT o.order_id, p.product_id, v.quantity, p.price
>FROM (VALUES ('anna.smith@example.com',   'Trail Running Shoes',   1),
>             ('anna.smith@example.com',   'Merino Base Layer',     2),
>             ('anna.smith@example.com',   'Headlamp',              1),
>             ('mark.jones@example.com',   '2-Person Tent',         1),
>             ('mark.jones@example.com',   'Daypack 30L',           1),
>             ('mark.jones@example.com',   'Trekking Poles',        2),
>             ('laura.brown@example.com',  'Hiking Boots',          1),
>             ('laura.brown@example.com',  'Waterproof Jacket',     1),
>             ('peter.miller@example.com', 'Trekking Backpack 65L', 1),
>             ('emma.wilson@example.com',  '4-Person Family Tent',  1))
>     AS v(email, product_name, quantity)
>JOIN customers c ON c.email = v.email
>JOIN orders o ON o.customer_id = c.customer_id
>JOIN products p ON p.name = v.product_name;
>
>
> ```

**Verify** each insert with `SELECT * FROM table_name;`

### Task 1.4: Practice UPDATE

Perform the following updates and verify each one:

1. Increase the price of all products in the Footwear category by 10%
2. Change customer #3's email to a new address
3. Update the status of order #2 from 'shipped' to 'delivered'
4. Set the stock of 'HydroFlask 1L' to 85
5. Add a description to any product that currently has NULL in description

> [!NOTE]
> ***Your SQL***
>
> ```sql
>
>1.
>UPDATE products
>SET price = ROUND(price * 1.10, 2)
>WHERE product_id IN (
>    SELECT pc.product_id
>    FROM product_categories pc
>    JOIN categories c ON c.category_id = pc.category_id
>    WHERE c.name = 'Footwear'
>);
>SELECT * FROM products;
>
>2.
>UPDATE customers
>SET email = 'laura.brown@newmail.com'
>WHERE customer_id = 3;
>SELECT * FROM customers;
>
>3. 
>UPDATE orders
>SET status = 'delivered'
>WHERE order_id = 2 AND status = 'shipped';
>SELECT * FROM orders;
>
>4.
>UPDATE products
>SET stock = 85
>WHERE name = 'HydroFlask 1L';
>SELECT * FROM products;
>
>5.
>UPDATE products
>SET description = 'Bright, durable headlamp for night hikes and camping.'
>WHERE name = 'Headlamp' AND description IS NULL;
>SELECT * FROM products;
>
>
> ```


### Task 1.5: Practice DELETE

1. Delete the most recently created order (and observe what happens to its order_items if you used CASCADE)
2. Try to delete a category that has products — what error do you get?

> [!NOTE]
> ***Your Answer***
> *(1. The Delete worked because it had ON DELETE CASCADE )*
> *(2. [23503] ERROR: update or delete on table "categories" violates foreign key constraint "product_categories_category_id_fkey" on table "product_categories" Detail: Key (category_id)=(1) is still referenced from table "product_categories".)*
>
>
>
>

3. Delete a customer who has no orders
>
>
>```sql
>DELETE FROM customers
>WHERE customer_id = (
>    SELECT c.customer_id
>    FROM customers c
>    WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id)
>    ORDER BY c.customer_id
>    LIMIT 1
>);
>
>SELECT * FROM customers;
>```

### Task 1.6: Practice ALTER TABLE

1. Add a column `phone VARCHAR(20)` to the customers table
2. Add a column `weight_grams INTEGER` to the products table
3. Add a CHECK constraint to ensure `weight_grams > 0` (allow NULL though — not all products have weight recorded yet)
4. Rename the `stock` column in products to `quantity_in_stock`

> [!NOTE]
> ***Your SQL***
>
> ```sql
>1.
>ALTER TABLE customers ADD COLUMN phone VARCHAR(20);
>
>2.
>ALTER TABLE products ADD COLUMN weight_grams INTEGER;
>
>3.
>ALTER TABLE products
>    ADD CONSTRAINT products_weight_grams_check CHECK (weight_grams > 0);
>
>4.
>ALTER TABLE products RENAME COLUMN stock TO quantity_in_stock;
>

>
>
> ```


---

## Exercise 2: Theory Review Questions

Answer the following questions in your own words using the answer fields below:

1. What does SQL stand for, and why was the language designed to look like English?

> [!NOTE]
> ***Your Answer***
>
> *(SQL stands for Structured Query Language. Its was designed to read like english so non programmers could analize and work without complexity to understand the language.)*
>
>
>
>

2. Explain the difference between DDL and DML. Give two example commands for each.

> [!NOTE]
> ***Your Answer***
>
> *(DDL (outside work) defines and changes the structure of the database and DML (inside work) works with the data inside the tables .)*
>
>
>
>

3. What is the difference between DCL and TCL? When would you use each?

> [!NOTE]
> ***Your Answer***
>
> *(DCL controls who can do stuff like `GRANT` and `REVOKE` its used to manage access. TCL controls the transactions like `BEGIN`, `COMMIT`, `ROLLBACK` it is used when several actions must fail or succeed.)*
>
>
>
>

4. Why must you create tables in a specific order? What determines that order?

> [!NOTE]
> ***Your Answer***
>
> *(A table with a foreign key can only reference a table that already exists. The order is determined by foreign key dependencies.)*
>
>
>
>

5. What is the difference between a column-level constraint and a table-level constraint? When *must* you use a table-level constraint?

> [!NOTE]
> ***Your Answer***
>
> *(A column-level constraint is written next to one column and applies only to that column, for example email VARCHAR(255) UNIQUE. A table-level constraint is written separately after the column list and can involve several columns.)*
>
>
>
>

6. Explain the difference between `DELETE FROM products;` and `TRUNCATE TABLE products;`. When would you prefer each?

> [!NOTE]
> ***Your Answer***
>
> *(`DELETE` removes rows one by one, can have a WHERE clause, is logged row by row, and does not reset the SERIAL counter. `TRUNCATE` empties the whole table at once, has no WHERE, is much faster, and can reset the counter with RESTART IDENTITY .)*
> *(Use `DELETE` when you want to remove only some rows. Use `TRUNCATE` to whipe a whole table quickly.)*
>
>
>

7. What does `ON DELETE CASCADE` do on a foreign key? Give a real-world scenario where it's appropriate and one where it would be dangerous.

> [!NOTE]
> ***Your Answer***
>
> *(`ON DELETE CASCADE` makes the database automatically delete child rows when the parent row is deleted. It is appropriate for `order_items` when an order is deleted, its line items have no meaning on their own. It is dangerous on something like orders.`customer_id` deleting one customer would silently delete all their orders )*
>
>
>
>

8. Why should you store `unit_price` in the `order_items` table instead of just looking it up from the `products` table?

> [!NOTE]
> ***Your Answer***
>
> *(Product prices change over time. If you only looked up the current price, an old order would suddenly show the new price.)*
>
>
>
>

9. What is the difference between SERIAL and GENERATED ALWAYS AS IDENTITY? Which would you use in a new project and why?

> [!NOTE]
> ***Your Answer***
>
> *(`SERIAL` is a PostgreSQL shortcut that creates a sequence and sets it as the column's default.`GENERATED ALWAYS AS IDENTITY` is the SQL-standard way, tied more cleanly to the column, and it rejects manually supplied ids unless you explicitly override it. I would use `GENERATED ALWAYS AS IDENTITY` because it is standard, safer, and the recommended modern approach.)*
>

10. Explain why `UPDATE products SET price = 9.99;` is dangerous. What steps should you take before running any UPDATE statement?

> [!NOTE]
> ***Your Answer***
>
> *(The update applies to every row, so every product in the table would get the price 9.99. You should write `SELECT` and `WHERE` first to see whih rows will be changed.You should run the update inside a transaction `(BEGIN)` so you can `ROLLBACK`;)*
>
>
>
>
---

## Exercise 3: SQL Writing Exercises

Write the SQL statements for each task in the **Your SQL** fields below. Verify by running them when ready.

### 3.1 CREATE TABLE

Write a CREATE TABLE statement for a `suppliers` table with the following columns:
- supplier_id (auto-incrementing primary key)
- company_name (required, max 200 characters, must be unique)
- contact_name (max 150 characters)
- email (max 255 characters, required, unique)
- phone (max 20 characters)
- country (max 100 characters, required, default 'Finland')

> [!NOTE]
> ***Your SQL***
>
> ```sql
> CREATE TABLE suppliers (
>    supplier_id SERIAL PRIMARY KEY,
>    company_name VARCHAR(200) NOT NULL UNIQUE,
>    contact_name VARCHAR(150),
>    email VARCHAR(255) NOT NULL UNIQUE,
>    phone VARCHAR(20),
>    country VARCHAR(100) NOT NULL DEFAULT 'Finland'
>);
>
>
> ```

### 3.2 CREATE TABLE with Foreign Key

Write a CREATE TABLE statement for a `product_reviews` table:
- review_id (auto-incrementing primary key)
- product_id (required, references products)
- customer_id (required, references customers)
- rating (required integer, must be between 1 and 5 inclusive)
- review_text (optional, unlimited length)
- created_at (required, defaults to current timestamp)

> [!NOTE]
> ***Your SQL***
>
> ```sql
> CREATE TABLE product_reviews (
>    review_id SERIAL PRIMARY KEY,
>    product_id INTEGER NOT NULL REFERENCES products(product_id),
>    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
>    rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
>    review_text TEXT,
>    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
>);
>
>
> ```

### 3.3 INSERT — Single Row

Write an INSERT statement to add a new category called 'Electronics' with description 'GPS devices, solar chargers, and tech gear'.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> INSERT INTO categories (name, description)
>   VALUES ('Electronics', 'GPS devices, solar chargers, and tech gear');
>
>
> ```

### 3.4 INSERT — Multiple Rows

Write a single INSERT statement that adds three new customers:
- Eero Lahtinen, eero.l@email.com
- Maria Salminen, maria.s@email.com
- Petri Kallio, petri.k@email.com

> [!NOTE]
> ***Your SQL***
>
> ```sql
> INSERT INTO customers (name, email)
>   VALUES ('Eero Lahtinen', 'eero.l@email.com'),
>       ('Maria Salminen', 'maria.s@email.com'),
>       ('Petri Kallio', 'petri.k@email.com');
>
>
> ```

### 3.5 INSERT with RETURNING

Write an INSERT statement that adds a new product called 'NorthStar GPS' priced at €229.99 with stock of 12 in category 'Electronics' (assume category_id = 6). Return the product_id and created_at.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> INSERT INTO products (name, price, quantity_in_stock)
>VALUES ('NorthStar GPS', 229.99, 12)
>   RETURNING product_id, created_at;
>
>
> ```

### 3.6 UPDATE — Simple

Write an UPDATE statement that changes the email of the customer with customer_id = 2 to 'mikko.korhonen@newmail.com'.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> UPDATE customers
>   SET email = 'mikko.korhonen@newmail.com'
>   WHERE customer_id = 2;
>
>
> ```

### 3.7 UPDATE — Expression

Write an UPDATE statement that reduces the stock of all products by 1 where the stock is currently greater than 0.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> UPDATE products
>   SET quantity_in_stock = quantity_in_stock - 1
>   WHERE quantity_in_stock > 0;
>
>
> ```

### 3.8 UPDATE — Multiple Columns

Write an UPDATE statement that changes order #3 to status 'cancelled' and sets a (hypothetical) cancelled_at timestamp to the current time. (Assume you've already added a cancelled_at column.)

> [!NOTE]
> ***Your SQL***
>
> ```sql
> UPDATE orders
>SET status = 'cancelled',
>    cancelled_at = CURRENT_TIMESTAMP
>WHERE order_id = 3;
>
>
> ```

### 3.9 DELETE — With Condition

Write a DELETE statement that removes all orders with status 'cancelled'.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> DELETE FROM orders
>   WHERE status = 'cancelled';
>
>
> ```

### 3.10 ALTER TABLE

Write the ALTER TABLE statements to:
a) Add a `discount_percent NUMERIC(5,2) DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100)` column to products
b) Drop the `description` column from categories
c) Add a composite unique constraint on (customer_id, product_id) in the product_reviews table (preventing a customer from reviewing the same product twice)

> [!NOTE]
> ***Your SQL***
>
> ```sql
> a)
>ALTER TABLE products
>    ADD COLUMN discount_percent NUMERIC(5,2) DEFAULT 0
>    CHECK (discount_percent >= 0 AND discount_percent <= 100);
>
> b)
>ALTER TABLE categories DROP COLUMN description;
>
> c)
>ALTER TABLE product_reviews
>    ADD CONSTRAINT product_reviews_customer_product_unique
>    UNIQUE (customer_id, product_id);
>
>
> ```

---

## Exercise 4: Error Diagnosis

Each of the following SQL statements contains one or more errors. Identify the error(s) and write the corrected version.

### 4.1

```sql
CREATE TABLE warehouses
    warehouse_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(100
);
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(after warehouse needs a `)` ; after `VARCHAR` there is also a missing `)`.)*
>
>
>


> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> CREATE TABLE warehouses (
>    warehouse_id SERIAL PRIMARY KEY,
>    name VARCHAR(100) NOT NULL,
>    city VARCHAR(100)
>);
>
>
> ```


### 4.2

```sql
INSERT INTO products (name, price, stock, category_id)
VALUES ("Alpine Sleeping Bag", 89.99, 20, 2);
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(A wrong string is used defining the name Alpine Sleeping Bag .)*
>
>
>


> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> INSERT INTO products (name, price, stock, category_id)
>   VALUES ('Alpine Sleeping Bag', 89.99, 20, 2);
>
>
> ```


### 4.3

```sql
CREATE TABLE shipments (
    shipment_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id)
    shipped_date DATE NOT NULL,
    carrier VARCHAR(100)
);
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(Theres a missing comma after `orders(order_id) `.)*
>
>
>


> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> CREATE TABLE shipments (
>    shipment_id SERIAL PRIMARY KEY,
>    order_id INTEGER REFERENCES orders(order_id),
>    shipped_date DATE NOT NULL,
>    carrier VARCHAR(100)
>);
>
>
> ```


### 4.4

```sql
UPDATE products
SET price = price * 0.9
SET stock = stock + 10
WHERE category_id = 3;
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(`SET` is used twice when you only need to put it once and missing comma after `0.9`.)*
>
>
>


> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> UPDATE products
>   SET price = price * 0.9,
>       stock = stock + 10
>   WHERE category_id = 3;
>
>
> ```


### 4.5

```sql
CREATE TABLE wishlists (
    wishlist_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    added_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (customer_id, product_id)
);
```

> [!NOTE]
> ***Error(s) Identified***
>
> *(There are two primary keys, only one is allowed so `whislist_id` is not needed .)*
>
>
>


> [!NOTE]
> ***Corrected SQL***
>
> ```sql
> CREATE TABLE wishlists (
>    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
>    product_id INTEGER NOT NULL REFERENCES products(product_id),
>    added_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
>    PRIMARY KEY (customer_id, product_id)
>);
>
>
> ```


---

## Submission Checklist

- [ ] All 5 TrailShop tables created successfully
- [ ] Sample data inserted (at least 5 categories, 5 customers, 10 products, 5 orders, 10 order items)
- [ ] UPDATE exercises completed and verified
- [ ] DELETE exercises completed and verified
- [ ] ALTER TABLE exercises completed and verified
- [ ] Theory review questions answered
- [ ] SQL writing exercises completed
- [ ] Error diagnosis completed with corrections
- [ ] All inline answer fields completed
