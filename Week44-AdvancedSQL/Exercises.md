# Week 44 — Advanced SQL: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

Continue building the TrailShop database. This week, use advanced query techniques to extract meaningful insights.

### Task 1.1: Subqueries

Write the following queries using subqueries:

1. Find all products priced above the average price (use a scalar subquery).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT name, price
> FROM products
> WHERE price > (SELECT AVG(price) FROM products);
>
>
> ```
2. Find all customers who have placed more than 2 orders (use a subquery in WHERE with IN).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT name
> FROM customers
> WHERE customer_id IN (SELECT customer_id FROM orders
> GROUP BY customer_id
> HAVING COUNT(*) > 2);
>
>
> ```
3. Show each product's name, price, and how much it differs from its category's average price (use a correlated subquery or derived table).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### Task 1.2: EXISTS

1. Use `EXISTS` to find all customers who have ordered at least one product from the "Footwear" category.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT customers.name
>FROM customers
>WHERE EXISTS(
>    SELECT 1
>    FROM orders
>    JOIN order_items ON order_items.order_id = orders.order_id
>    JOIN products ON products.product_id = order_items.product_id
>    JOIN product_categories ON product_categories.product_id = products.product_id
>    JOIN categories ON categories.category_id = product_categories.category_id
>    WHERE orders.customer_id = customers.customer_id
>      AND categories.name IN ('Footwear')
>);
>
>
> ```
2. Use `NOT EXISTS` to find all products that have never been ordered.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT products.name
> FROM products
> WHERE NOT EXISTS (
>    SELECT 1
>    FROM order_items
>    WHERE order_items.product_id = products.product_id
>);
>
>
> ```

### Task 1.3: CASE Expressions

1. Add a `price_tier` column to a product query that labels products as 'Budget' (< €25), 'Standard' (€25–€74.99), or 'Premium' (≥ €75).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT products.name,
>       products.price,
>       CASE
>           WHEN price < 25 THEN 'Budget'
>           WHEN price < 74.99 THEN 'Standard'
>           WHEN price >= 75 THEN 'Premium'
>        END AS price_tier
> FROM products;
>
>
> ```
2. Write a query that counts orders by status using CASE inside COUNT.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT
>    COUNT(*) AS total_orders,
>    COUNT(CASE WHEN status = 'shipped' THEN 1 END) AS shiped,
>    COUNT(CASE WHEN status = 'paid' THEN 1 END) AS paid
> FROM orders;
>
>
> ```

### Task 1.4: Create Three Views

Create the following views in your TrailShop database:

1. `v_product_catalog` — Shows product name, category name, price, and stock status ('In Stock', 'Low Stock' for < 5 units, 'Out of Stock' for 0).
2. `v_customer_summary` — Shows each customer's full name, email, number of orders, and total spent.
3. `v_monthly_revenue` — Shows revenue grouped by month.

Verify each view works with a simple SELECT.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> CREATE VIEW v_product_catalog AS
> SELECT
>    products.name AS product_name,
>    categories.name AS category_name,
>    products.price,
>    products.quantity_in_stock,
>    CASE
>        WHEN products.quantity_in_stock = 0 THEN 'Out of Stock'
>        WHEN products.quantity_in_stock < 5 THEN 'Low Stock'
>        ELSE 'In Stock'
>    END AS available
> FROM products
> JOIN product_categories ON product_categories.product_id =  products.product_id
> JOIN categories ON categories.category_id = product_categories.category_id;
>
> CREATE VIEW v_customer_summary AS
> SELECT
>    customers.name AS full_name,
>    customers.email,
>    COUNT(DISTINCT orders.order_id) AS order_count,
>    COALESCE(SUM(order_items.quantity * order_items.unit_price), 0) AS total_spent
>FROM customers
>LEFT JOIN orders ON orders.customer_id = customers.customer_id
>LEFT JOIN order_items ON order_items.order_id = orders.order_id
>GROUP BY customers.customer_id, customers.customer_id, customers.email ;
>
>CREATE VIEW v_monthly_revenue AS
>    SELECT
>    DATE_TRUNC('month', order_date) AS month,
>    SUM(order_items.quantity * order_items.unit_price) AS revenue
>FROM orders
>JOIN order_items ON order_items.order_id = orders.order_id
>GROUP BY month;
>
>
> ```

### Task 1.5: UNION

1. Write a UNION query that combines product names from two different categories into one list (without duplicates).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT products.name
> FROM products
> JOIN product_categories ON product_categories.product_id = products.product_id
> JOIN categories ON categories.category_id = product_categories.category_id
> WHERE categories.name = 'Footwear'
>
> UNION
> 
> SELECT products.name
> FROM products
> JOIN product_categories ON product_categories.product_id = products.product_id
> JOIN categories ON categories.category_id = product_categories.category_id
> WHERE categories.name = 'Clothing'
>
>
> ```
2. Write a query using EXCEPT to find categories that have products but no orders.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT categories.name
> FROM categories
> JOIN product_categories ON product_categories.category_id = categories.category_id
>
> EXCEPT
>
> SELECT categories.name
> FROM categories
> JOIN product_categories ON product_categories.category_id = categories.category_id
> JOIN order_items ON order_items.product_id = product_categories.product_id;
>
>
> ```

### Task 1.6: CTE

1. Write a CTE that calculates total revenue per category, then use it to find categories with above-average revenue.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> WITH
> category_revenue AS (
>    SELECT
>        pc.category_id,
>        SUM(oi.quantity * oi.unit_price) AS revenue
>    FROM product_categories pc
>    JOIN order_items oi ON oi.product_id = pc.product_id
>    GROUP BY pc.category_id
>),
> avg_category_revenue AS (
>    SELECT AVG(revenue) AS avg_revenue
>    FROM category_revenue
>)
> SELECT
>    cat.name AS category_name,
>    cr.revenue,
>    ROUND(acr.avg_revenue, 2) AS avg_revenue
> FROM category_revenue cr
> JOIN categories cat ON cat.category_id = cr.category_id
> CROSS JOIN avg_category_revenue acr
> WHERE cr.revenue > acr.avg_revenue
> ORDER BY cr.revenue DESC;
>
>
> ```
2. Write a query with two CTEs: one for customer order counts, another for average order count, then find "top customers" (above average).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> WITH
> customer_order_counts AS (
>    SELECT
>        c.customer_id,
>        c.name,
>        COUNT(o.order_id) AS order_count
>    FROM customers c
>    LEFT JOIN orders o ON o.customer_id = c.customer_id
>    GROUP BY c.customer_id, c.name
> ),
> avg_order_count AS (
>    SELECT AVG(order_count) AS avg_orders
>    FROM customer_order_counts
> )
> SELECT
>    coc.name,
>    coc.order_count,
>    ROUND(aoc.avg_orders, 2) AS avg_orders
> FROM customer_order_counts coc
> CROSS JOIN avg_order_count aoc
> WHERE coc.order_count > aoc.avg_orders
> ORDER BY coc.order_count DESC;
>
>
> ```

---

## 2. Theory Review Questions

Answer these in your own words (2–4 sentences each):

1. What is the difference between an INNER JOIN and a LEFT JOIN? When would you choose LEFT JOIN?

> [!NOTE]
> ***Your Answer***
>
> *(`INNER JOIN` returns only rows that have a match in both tables. `LEFT JOIN` returns all rows from the left table, and if there is no match the right side columns are NULL. I would choose `LEFT JOIN` when I don't want to lose rows.)*
>
>
>
>


2. Explain what a correlated subquery is. Why can it be slower than an uncorrelated subquery?

> [!NOTE]
> ***Your Answer***
>
> *(A correlated subquery is a subquery that uses a value from the outer query, so it runs again for every row of the outer query, like a loop. For example comparing each product's price to the average of its own category. An uncorrelated subquery runs only once, so the correlated one can be slow on big tables.)*
>
>
>
>


3. What is the difference between `IN` and `EXISTS`? When might you prefer EXISTS?

> [!NOTE]
> ***Your Answer***
>
> *(`IN` checks if a value is in a list or in the result of a subquery. `EXISTS` only checks if the subquery returns at least one row, it doesn't care about the values. `EXISTS` can stop at the first match, so I would prefer it when the subquery result is big or the condition is complex.)*
>
>
>
>


4. What rules must two SELECT statements follow to be combined with UNION?

> [!NOTE]
> ***Your Answer***
>
> *(Both `SELECT` statements must return the same number of columns. The columns in the same position must have compatible data types.)*
>
>
>
>


5. Explain the difference between UNION and UNION ALL. When would you use each?

> [!NOTE]
> ***Your Answer***
>
> *(`UNION` combines the results and removes duplicates. `UNION ALL` keeps all rows including duplicates and is faster because it skips the duplicate check. I would use `UNION` when a row could appear twice and I want it only once. `UNION ALL` when there are no duplicates or I want to count them.)*
>
>
>
>


6. What is a view? Name two advantages of using views.

> [!NOTE]
> ***Your Answer***
>
> *(A view is a saved query with a name that works like a virtual table, it doesn't store the data itself. Advantage 1: it simplifies complex queries. Advantage 2: security, you can give a user access to the view instead of the full table with sensitive columns.)*
>
>
>
>
7. What does `WITH CHECK OPTION` do on an updatable view?

> [!NOTE]
> ***Your Answer***
>
> *(`WITH CHECK OPTION` blocks `INSERT` or `UPDATE` through the view if the new row would not be visible in that view anymore because of its `WHERE`.)*
>
>
>
>


8. How does a CTE differ from a derived table (subquery in FROM)? What's the main advantage of CTEs?

> [!NOTE]
> ***Your Answer***
>
> *(A derived table is a subquery written inside `FROM` and it must have an alias. A CTE is written at the start with `WITH` and gets a name, so the query reads from top to bottom step by step. The main advantage is readability, and a CTE can be used multiple times in the same query.)*
>
>
>
>

---

## 3. Query Writing Exercises

Write SQL queries for the following (use the TrailShop schema):

1. Find the most expensive product in each category using a correlated subquery.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
2. List all customers along with their most recent order date. Include customers who have never ordered (show NULL).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
3. Find categories where the total revenue exceeds €500 (use a subquery or CTE).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
4. Write a query that shows each order's total and whether it's 'Small' (< €50), 'Medium' (€50–€149.99), or 'Large' (≥ €150).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
5. Create a view called `v_never_ordered` that shows products that have never appeared in an order_item.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
6. Use EXCEPT to find products that exist in the products table but have never been ordered.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
7. Write a CTE that ranks categories by number of products, then selects only the top 3.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
8. Use `NOT EXISTS` to find categories that have no products.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
9. Write a query using CASE that creates a "customer loyalty" label: 'New' (1 order), 'Regular' (2–5 orders), 'VIP' (6+ orders).
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
10. Use GREATEST to find the higher value between each product's current price and the average price of all products in its category.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
11. Write a query using COALESCE that shows order shipping dates, substituting 'Not yet shipped' for NULL values.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```
12. Use a scalar subquery in the SELECT clause to show each product alongside the total number of times it has been ordered.
> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

---

## 4. Query Comparison Exercise

For each pair of queries below, both produce the same result. Explain which approach you'd prefer and why (consider readability, performance, and maintainability).
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### Pair A: Finding customers with orders

**Version 1 — Subquery with IN:**
```sql
SELECT first_name, last_name
FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);
```

**Version 2 — JOIN:**
```sql
SELECT DISTINCT c.first_name, c.last_name
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id;
```

### Pair B: Finding products never ordered

**Version 1 — LEFT JOIN + IS NULL:**
```sql
SELECT p.product_name
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.product_id
WHERE oi.order_item_id IS NULL;
```

**Version 2 — NOT EXISTS:**
```sql
SELECT p.product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.product_id
);
```

### Pair C: Customer total spending

**Version 1 — Derived table (subquery in FROM):**
```sql
SELECT sub.full_name, sub.total_spent
FROM (
    SELECT c.first_name || ' ' || c.last_name AS full_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
) sub
WHERE sub.total_spent > 200;
```

**Version 2 — CTE:**
```sql
WITH customer_totals AS (
    SELECT c.first_name || ' ' || c.last_name AS full_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT full_name, total_spent
FROM customer_totals
WHERE total_spent > 200;
```

For each pair, discuss: readability, performance (in PostgreSQL), and when you'd choose one over the other.
> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
