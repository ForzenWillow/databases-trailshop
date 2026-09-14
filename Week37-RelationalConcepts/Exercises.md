# Week 37 — Exercises & Project Task

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 37 Theory material. Complete all sections.

---

## Part 1: TrailShop Project Task

### Task 1: Identify Keys

Using the `products`, `categories`, and `customers` tables shown in Section 2 of this week's Theory material, answer:

1. What is the primary key of the `products` table? Why is it a good choice?

> [!NOTE]
> ***Your Answer***
>
> *( The primary key in products is - product_id since its a unique row you can see in other table like categories: category_id and customers: customer_id. Since its a unique row it follow a way to store unique things that are important and cant be Null and should be stable. Thats what idetifies the row in the table.)*
>
>
>
>


2. What is the primary key of the `categories` table?

> [!NOTE]
> ***Your Answer***
>
> *(In categories its : category_id.)*
>
>
>
>


3. What is the foreign key in the `products` table? What does it reference?

> [!NOTE]
> ***Your Answer***
>
> *(A Foreign key in the prducts table is : category_id . It references the primary Key and its rows like the product_id 101 and category_id is 1 and so on.)*
>
>
>
>


4. Is `name` in `products` a candidate key? Under what assumption? What would make it unsuitable as a primary key?


> [!NOTE]
> ***Your Answer***
>
> *(Yes 'name' in 'products' is a candidate key in this situation. Since every single name in the collumn is unique. It would make it unsuitable when there would be repetition then it wouldnt be considered a primary key since for it to be PK it would need uniqueness  .)*
>
>
>
>
5. Give an example of a **superkey** for the `products` table that is NOT a candidate key. Explain why it's not minimal.

> [!NOTE]
> ***Your Answer***
>
> *(Not a Superkey in 'products' is the 'price',stock_quantity nad category_id, since there could be items in the table that have the same price, same category id or same stock numbers. For the superkey its manditory to have an attribute that is uniquely indentified .)*
>
>
>
>

6. Give an example of a **composite key** using a hypothetical `order_items` table. Explain why neither column alone would be sufficient.

> [!NOTE]
> ***Your Answer***
>
> *(An example would be order_id and product_id, on theyr own they are not uniques since a lot of id appear twice but when you put them together they are unique, because each product appears at most once.)*
>
>
>
>

7. Is `email` in `customers` a candidate key? What makes it different from `customer_id` as a PK choice? *(See Section 6.9 on natural vs surrogate keys.)*

> [!NOTE]
> ***Your Answer***
>
> *(Yes 'email' in 'customers' can be a candidate key since there cant be two users sahring the same email address. Whats differet is that 'customer_id' is a surrogated key while 'email' is a natural key . Natural Keys can change an may be long and surrogated keys are more stable and have only single integers)*
>
>
>
>

### Task 2: Define Business Rules

List **5 business rules** for TrailShop. For each rule, specify:
- The rule in plain English
- Which constraint type(s) would enforce it
- Which table and column the constraint applies to
- The SQL syntax for the constraint

Example:

| Business Rule | Constraint Type | Table.Column | SQL |
|---|---|---|---|
| Every product must have a price greater than zero | CHECK | products.price | `CHECK (price > 0)` |
| ... | ... | ... | ... |

Think about rules for customers, orders, and categories — not just products.

> [!NOTE]
> ***Your Answer***
>
> *( | Every single customers emails must be unique  | UNIQUE | email | `email VARCHAR(255) UNIQUE` |   | Every product has to have a name | NOT NULL | product_name | `name VARCHAR(100) NOT NULL` | | Orders must have statuses, where are they right now | CHECK | order_status | `CHECK (status IN ('pending','shipped','delivered','cancelled'))` | | Every product mus belong to a category of items | NOT NULL + FK | products.category_id | `category_id INTEGER NOT NULL REFERENCES categories` |  | Stock cannot be negative | CHECK | products.stock_quantity | `CHECK (stock_quantity >= 0)` | )*
>
>
>
>

### Task 3: Integrity Violations

For each SQL statement below, predict whether it will **succeed** or **fail**. If it fails, explain which integrity rule or constraint is violated and what error message you'd expect. Assume the schema from Section 9.8 of the Theory material.

```sql
-- Statement A
INSERT INTO categories (category_id, category_name)
VALUES (NULL, 'Cycling');

-- Statement B
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (109, 'AeroLite Tent', 279.00, 10, 2);

-- Statement C
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (110, 'BudgetBoots', -5.00, 25, 1);

-- Statement D
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (103, 'Duplicate Shoes', 99.99, 5, 3);

-- Statement E
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (111, 'CloudWalker Sandals', 65.00, 40, 10);

-- Statement F
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (112, NULL, 89.99, 20, 1);

-- Statement G
INSERT INTO products (product_id, name, price, stock_quantity, category_id)
VALUES (113, 'LightStep Shoes', 149.00, -3, 1);

-- Statement H
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (1001, 101, 0, 189.50);
```

> [!NOTE]
> ***Your Answer***
>
> *(  A - Will **fail** because the category_id is set to NULL and it cannot be NULL, especially if its a PK.  )*
>
> *( B -  It will **succeed** )*
>
> *( C -  It will **fail** because `price` cannot be lower than 0)*
>
> *( D -  It will **fail** because there is already a product_id with 103 and it is a duplicate so it wont go through)*
>
> *( E - it will **succeed** )*
>
> *( F - it will **fail** because the name cannot be NULL )*
>
> *( G - it will **fail** because stock_quantity cannot be lower than 0)*
>
> *( H - it will **fail** because quantity cannot be 0)*

### Task 4: Foreign Key Actions

Consider the following scenario using the schema from Theory Section 9.8:

1. You want to delete category 2 ("Camping") from the `categories` table. Products 102 and 106 reference this category. What happens with:
   - `ON DELETE RESTRICT`?
   - `ON DELETE CASCADE`?
   - `ON DELETE SET NULL`? (Assume `category_id` in `products` allows NULL for this question)

2. Which foreign key action would you recommend for the TrailShop `products.category_id` → `categories.category_id` relationship? Justify your choice in 2–3 sentences.

> [!NOTE]
> ***Your Answer***
>
> *( 1. RESTRICT will get blocked because products 102 and 106 still use it as a reference so youd have to delete those products to delete the category. CASCADE will delete the table and all of the referencing rows. SET NULL the category 2 will get deleted and those products that have referenced it will be set to NULL  .)*
> *(2. Id recommend RESTRICT so if someone wants to delete the category 2 they would have to first delete all the products or rows that reference that table for safety reasons. RESTRICT is better since if we would use NULL we would violate NOT NULL statement and CASCADE would just delete everything before even thining twice. )*
>
>
>




---

## Part 2: Theory Review Questions

Answer each question in 2–4 sentences unless otherwise specified. Reference the Theory material sections as needed.

### Short-Answer Questions

**Q1.** Define the following terms in your own words: relation, tuple, attribute, domain. Give one TrailShop example for each.

> [!NOTE]
> ***Your Answer***
>
> *( Relation - Is a Table with sets of Rows and columns, and in those rows/columns there are values/information. For example in Trailshop - Categories, products and customers are tables and in them there are rows/columns of information.)*
>
> *( Tuple - Is a row and in each row there are a set of values. For example in Trailshop : (102, 'TrailMaster X4 Tent', 249.99, 15, 2). This tuple/row says that this Product is named TrailMaster X4 Tent, costs 249.99, has 15 units in stock, and belongs to category 2. .)*
>
> *( Attribute - Is a column that has a distinct name. For example in Trailshop we have: product_id, name, category_id, order_date and exc.)*
>
> *( Domain - Is a set of values with particular attributes. For example in Trailshop product_id has Positive integers value and price has positive decimal numbers.)*

*(See Sections 2 and 3 of this week's Theory material.)*

**Q2.** What makes a candidate key different from a primary key? Can a table have more than one candidate key?


> [!NOTE]
> ***Your Answer***
>
> *( Candidate key could be any column that could identify a row uniquely and Primary key is just a candidate key thay we pick to be like an official identifier fot the table In Trailshop the candidate key could be product_id and name if its guaranteed to have unique names  .)*
>
>
>
>

*(See Section 6 of this week's Theory material.)*

**Q3.** Explain entity integrity in your own words. Why can't a primary key be NULL?


> [!NOTE]
> ***Your Answer***
>
> *( Entity integrity is like bones of the structure if there wouldnt be no PK it you wouldnt be able to relaibly find, update or delete things. The same idea goes if the priamry key was NULL you wouldnt be able to relaibly do things .)*
>
>
>
>

*(See Section 8.1 of this week's Theory material.)*

**Q4.** What happens when referential integrity is violated? Give a concrete TrailShop example — show the SQL statement and the expected error.

> [!NOTE]
> ***Your Answer***
>
> *( If referential intefrity is violated there will be orphan records and we will try to address something that does not exist. )*
>
> *(`INSERT INTO products (product_id, name, price, stock_quantity, category_id) VALUES (109, 'Ghost Product', 59.99, 5, 99);`)*
> *(`ERROR:  insert or update on table "products" violates foreign key constraint "products_category_id_fkey" DETAIL:  Key (category_id)=(99) is not present in table "categories".`)*
>
>

*(See Section 8.2 of this week's Theory material.)*

**Q5.** Explain the difference between a surrogate key and a natural key. Give an example of each for a `books` table in a library database.

> [!NOTE]
> ***Your Answer***
>
> *( Surrogated key is an artificial key and its purpose is to uniquely identify rows and Natrual key is a key that is drawn from the data itself and not generated, to use as an alternate key when it is appropriate.)*
>
>
>
>

*(See Section 6.8–6.9 of this week's Theory material.)*

**Q6.** What is a NULL value? Why is `WHERE price = NULL` wrong? What should you write instead?


> [!NOTE]
> ***Your Answer***
>
> *( NULL value means that the value is unknown or just an empty string , theres nothing. Because if you use NULL to calculate the price or try to add to it a price for example `SELECT NULL + 67;` it will result to NULL. You should use `IS NULL` or `IS NOT NULL`.)*
>
>
>
>

*(See Section 7 of this week's Theory material.)*

**Q7.** What is a junction table? When is it needed? Give an example.

> [!NOTE]
> ***Your Answer***
>
> *( Junction table is a bridge between A table and B table since they cannot be represented directly with a foreign key. It is needed when you need to clearly represent both tables like `order_items` in Trailshop its a junction between orders and products tables.)*
>
>
>
>

*(See Section 12.3 of this week's Theory material.)*

**Q8.** Describe the three types of relationships (1:1, 1:N, M:N). For each, give one TrailShop example.

> [!NOTE]
> ***Your Answer***
>
> *( 1:1 One-to-one is when one row in a table A also relates to exactly one row on a table B.)*
>
> *( products (1) ──── (1) product_details )*
>
>
> *( 1:N One-to-Many is when a table A relates to many rws in a table B.)*
>
>*(categories (1) ──── (N) products)*
>*(customers  (1) ──── (N) orders)*
>*(orders     (1) ──── (N) order_items)*
>
>
>*( M:M Many-to-Many is when many rows in table A relate to many rows in table B.)*
>
>*( products (M) ──── (N) tags)*
>

*(See Section 12 of this week's Theory material.)*

**Q9.** What is the difference between `ON DELETE CASCADE` and `ON DELETE RESTRICT`? When would you use each?


> [!NOTE]
> ***Your Answer***
>
> *( Cascade deletes everything, the row and what references it and restrict deletes the row if there are no referecing rows, if there there will be an error.)*
>
> *(cascade would be used when theres no meaning if its gone and restrict when there are independent values so everything wouldnt go)*
>
>

*(See Section 10 of this week's Theory material.)*

**Q10.** Explain what "atomic entries" means in the context of relation properties. Give an example of a violation.

> [!NOTE]
> ***Your Answer***
>
> *( Its an invisable entrie that holds one row, one column and adds into one value. An example would be putting multiple categories in one cell, like putting Footware and Hiking into categories)*
>
>
>
>

*(See Section 5.3 of this week's Theory material.)*

### True/False

For each statement, write **True** or **False** and correct any false statements.

1. **FALSE** A superkey is always a candidate key.
2. **TRUE** A primary key can consist of more than one column.
3. **FALSE** NULL = NULL evaluates to TRUE in SQL.
4. **FALSE** A foreign key must always be NOT NULL.
5. **TRUE** Referential integrity ensures that every FK value matches an existing PK value (or is NULL).
6. **FALSE** The degree of a relation is the number of rows.

### Matching Exercise

Match each term (1–12) with its definition (A–L).

| # | Term |
|---|---|
| 1 + | Superkey |
| 2 + | Candidate key |
| 3 + | Composite key |
| 4 + | Foreign key |
| 5 + | Alternate key |
| 6 + | Surrogate key |
| 7 + | Natural key |
| 8 + | Orphan record |
| 9 + | Domain |
| 10 + | Junction table |
| 11 + | Cardinality |
| 12 + | COALESCE |

| Letter | Definition |
|---|---|
| A + | The set of all permitted values for an attribute |
| B + | A key composed of two or more attributes |
| C + | A row whose FK references a non-existent PK — forbidden by referential integrity |
| D + | An artificial key with no business meaning (e.g., auto-generated ID) |
| E + | A candidate key not chosen as the primary key |
| F + | Any set of attributes that uniquely identifies every tuple |
| G + | A minimal superkey — no attribute can be removed without losing uniqueness |
| H + | A column <-?? that references the primary key of another table |
| I + | The number of tuples (rows) in a relation |
| J + | A key drawn from real-world data with business meaning |
| K + | A table implementing a many-to-many relationship |
| L + | A SQL function that returns the first non-NULL argument |


> [!NOTE]
> ***Your Answers***
>
> | # | Your Match |
> |---|---|
> | 1 | F |
> | 2 | G |
> | 3 | B |
> | 4 | H |
> | 5 | E |
> | 6 | D |
> | 7 | J |
> | 8 | C |
> | 9 |  A |
> | 10 | K |
> | 11 | I |
> | 12 | L |
>

---

## Part 3: SQL Practice — Constraints in Action

These exercises test your understanding of constraints. You do NOT need to run these in PostgreSQL (but you may if you'd like to verify your answers).

### Exercise 3.1: Predict the Outcome

Given the following table definitions:

```sql
CREATE TABLE departments (
    dept_id   INTEGER      PRIMARY KEY,
    dept_name VARCHAR(50)  NOT NULL UNIQUE
);

CREATE TABLE employees (
    emp_id    INTEGER       PRIMARY KEY,
    name      VARCHAR(100)  NOT NULL,
    salary    NUMERIC(10,2) NOT NULL CHECK (salary >= 0),
    dept_id   INTEGER       NOT NULL REFERENCES departments(dept_id)
);
```

Assume these rows already exist:

```sql
INSERT INTO departments VALUES (1, 'Engineering'); <-- this will pass
INSERT INTO departments VALUES (2, 'Marketing'); <-- this will pass
INSERT INTO employees VALUES (100, 'Alice', 75000, 1); <-- this will pass
INSERT INTO employees VALUES (101, 'Bob', 65000, 2); <-- this will pass
```

For each statement below, predict: **SUCCESS** or **FAIL**? If fail, name the violated constraint.

```sql
-- 1
INSERT INTO employees VALUES (102, 'Carol', 70000, 1); "**SUCCESS**"

-- 2
INSERT INTO employees VALUES (103, 'Dan', -5000, 1); "**FAIL** salary cant be minus"

-- 3
INSERT INTO employees VALUES (100, 'Eve', 80000, 2); "**FAIL** emp_id must be unique"

-- 4
INSERT INTO employees VALUES (104, 'Frank', 60000, 5); " **FAIL** theres no department 5"

-- 5
INSERT INTO departments VALUES (3, 'Engineering'); "**FAIL** name has to be UNIQUE"

-- 6
INSERT INTO employees VALUES (105, NULL, 55000, 2); "**FAIL** name cant be NULL"

-- 7
DELETE FROM departments WHERE dept_id = 1; "**FAIL** it fails because of the REFERENCE in dept_id"

-- 8
INSERT INTO employees VALUES (106, 'Grace', 0, 2); "**SUCCESS**"
```

### Exercise 3.2: Write the Constraints

Given these business rules for a **bookstore database**, write the `CREATE TABLE` statements with appropriate constraints:

1. Every book has a unique ISBN (13 characters), a title (required), a price (must be positive), and a publication year. +
2. Every author has an ID, a first name (required), and a last name (required). +
3. A book can have multiple authors, and an author can write multiple books.
4. Every book belongs to exactly one genre. Genres have an ID and a unique name.
5. Publication year must be between 1450 and the current year. +

*(Hint: you'll need at least 4 tables, including a junction table for the M:N relationship.)*

>[!NOTE]
> ***Your Answers***
**(1.)**
```sql
CREATE TABLE books(
    book_id    INTEGER     PRIMARY KEY,
    isbn          VARCHAR(13)      NOT NULL UNIQUE,
    title         VARCHAR(100)     NOT NULL,
    price         NUMERIC(10,2)    NOT NULL CHECK (price > 0),
    year          INTEGER          NOT NULL CHECK (year BETWEEN 1450 AND 2026),
    genre_id      INTEGER          NOT NULL REFERENCES genre(genre_id)
);
```
**(2.)**
```sql
CREATE TABLE authors (
    author_id     INTEGER       PRIMARY KEY,
    first_name    VARCHAR(100)  NOT NULL,
    last_name     VARCHAR(100)  NOT NULL
);
```

**(3.)**
```sql
CREATE TABLE book_tags(
    author_id     INTEGER     REFERENCES authors(author_id),
    book_id       INTEGER     REFERENCES books(book_id),
    PRIMARY KEY (author_id, book_id)
);
```

**(4.)**
```sql
CREATE TABLE genre (
    genre_id        INTEGER     PRIMARY KEY,
    genre_name      VARCHAR(100)    NOT NULL UNIQUE
);
```
---

## Part 4: Design Exercise — Library System

A small public library needs a database. Here is a description of their requirements:

> The library has a collection of **books**. Each book has an ISBN, a title, a publication year, and belongs to one genre (Fiction, Non-Fiction, Science, History, etc.). The library may own multiple **copies** of the same book — each copy has a unique barcode sticker.
>
> The library has registered **members**. Each member has a member number, name, email, and phone. Members can **borrow** copies. Each borrowing records which member borrowed which copy, the borrow date, the due date, and the return date (NULL if not yet returned).
>
> **Rules:**
> - A member can borrow at most 5 copies at any given time.
> - The due date is always 14 days after the borrow date.
> - A copy cannot be borrowed if it's currently not returned (return_date IS NULL).

### Your Tasks

1. **Identify the tables** you would need (list them with their columns).
2. **Identify the primary key** for each table. Are they surrogate or natural keys? Justify your choices.
3. **Identify all foreign keys** and the tables they reference.
4. **Identify any candidate keys** beyond the primary key (alternate keys).
5. **List the business rules** from the description and map each to a constraint type. Which rules cannot be enforced by simple constraints?


> [!NOTE]
> ***Your Answer***
> 
> ```sql
> CREATE TABLE books(
>   book_id     INTEGER           PRIMARY KEY,
>   isbn        VARCHAR(13)       NOT NULL UNIQUE,
>   title       VARCHAR(100)      NOT NULL,
>   publication_year    INTEGER   NOT NULL CHECK (publication_year BETWEEN 1450 AND 2026),
>   genre_id     INTEGER          NOT NULL REFERENCES genre(genre_id)
>); 
> ```
>
>```sql
> CREATE TABLE genre (
>   genre_id    INTEGER     PRIMARY KEY,
>   genre_name  VARCHAR(50)     NOT NULL UNIQUE
>);
>
>```
>
>```sql
> CREATE TABLE copies (
>   copy_id    INTEGER     PRIMARY KEY,
>   book_id    INTEGER     NOT NULL REFERENCES books(book_id),
>   barcode_sticker   VARCHAR(50)       NOT NULL UNIQUE
>);
>```
> 
>```sql
> CREATE TABLE members (
>   member_id       INTEGER         PRIMARY KEY,
>   member_number   INTEGER         NOT NULL UNIQUE,
>   member_name     VARCHAR(50)     NOT NULL,
>   member_email    VARCHAR(255)    NOT NULL UNIQUE,
>   phone_number    VARCHAR(20)     NOT NULL 
>);
>
>```
>```sql
> CREATE TABLE borrowings (
>   borrowing_id    INTEGER     PRIMARY KEY,
>   copy_id         INTEGER     REFERENCES copies(copy_id),
>   member_id       INTEGER     REFERENCES members(member_id),
>   borrow_date     DATE        NOT NULL,
>   due_date        DATE        NOT NULL,
>   return_date     DATE  
>);
>
>```
>
>
> *(1-3 are displayed in the upper level , 4. Alternate keys could be : books.ibsn; genre.genre_name; )*
> *(5. [1] A Copy cant be borrowed if its still not returned - It has to look at other rows before it can allow to INSERT. [2] Due Date = Borrow date + 14 days - Can be a CHECK comparing two columns . [3] A member can have at most 5 unreturned books - Needs to count rows across the table.)*
>*(Rules that cannit be enforced : 1. and 3.)*
>
>
6. **Write the CREATE TABLE statements** for at least the `books`, `copies`, and `borrowings` tables with full constraints.

*(Check the upper task)*

## Submission Checklist

- [ ] Task 1: Key identification answers (Part 1)
- [ ] Task 2: Business rules table with 5 rules (Part 1)
- [ ] Task 3: Integrity violation predictions with explanations (Part 1)
- [ ] Task 4: Foreign key action analysis (Part 1)
- [ ] Theory Review Questions answered (Part 2)
- [ ] SQL Practice — constraint predictions and bookstore CREATE TABLE (Part 3)
- [ ] Library System design exercise (Part 4)
