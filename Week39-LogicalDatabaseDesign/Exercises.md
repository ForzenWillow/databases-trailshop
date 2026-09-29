# Week 39 — Logical Database Design: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 39 Theory material. Refer to the theory sections indicated in brackets when you need help.

---

## Exercise 1: TrailShop Project Task — Build the Schema

**Goal:** Convert the TrailShop ER diagram (from Week 38) into a complete PostgreSQL relational schema.

### Instructions

Write `CREATE TABLE` statements for all five TrailShop tables:

1. `categories`
2. `customers`
3. `products`
4. `orders`
5. `order_items`

### Requirements

For each table, you must:

- Choose appropriate PostgreSQL data types for every column (justify at least 3 choices in writing)
- Define primary keys (surrogate or composite as appropriate)
- Define foreign keys with explicit `ON DELETE` and `ON UPDATE` actions (justify each choice)
- Add `NOT NULL`, `UNIQUE`, `CHECK`, and `DEFAULT` constraints where appropriate
- Create tables in the correct dependency order
- Follow the naming conventions from Theory Section 8

### Deliverables

1. A single `.sql` file with all five `CREATE TABLE` statements (executable in PostgreSQL)
2. A short written document (1–2 pages) containing:
   - Justification for 3 data type choices (e.g., why `NUMERIC(10,2)` for price instead of `REAL`)
   - Justification for each FK action choice (e.g., why CASCADE on `order_items.order_id`)
   - One design decision you made that wasn't specified in the requirements (e.g., whether shipping address is optional)

### Bonus Challenge

After creating the tables, insert sample data:
- At least 5 categories
- At least 8 products (across at least 3 categories)
- At least 3 customers
- At least 4 orders (across at least 2 customers)
- At least 10 order items

Verify that your constraints work by attempting at least 2 invalid inserts and showing the error messages.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> 
>
>CREATE TABLE categories (
>    category_id    SERIAL PRIMARY >KEY,
>    category_name  VARCHAR(100) >NOT NULL UNIQUE,
>    description    TEXT
>);
>
>CREATE TABLE customers (
>    customer_id    SERIAL PRIMARY >KEY,
>    first_name     VARCHAR(100) >NOT NULL,
>    last_name      VARCHAR(100) >NOT NULL,
>    email          VARCHAR(255) >NOT NULL UNIQUE,
>    phone          VARCHAR(30),
>    street         VARCHAR(255) >NOT NULL,
>    city           VARCHAR(100) >NOT NULL,
>    postal_code    VARCHAR(20)  >NOT NULL,
>    country        VARCHAR(100) >NOT NULL DEFAULT 'Finland',
>    registered_at  TIMESTAMPTZ  >NOT NULL DEFAULT NOW()
>);
>
>CREATE TABLE products (
>    product_id      SERIAL PRIMARY >KEY,
>    name            VARCHAR(255) >NOT NULL,
>    description     TEXT,
>    price           NUMERIC(10, 2) >NOT NULL CHECK (price > 0),
>    weight_kg       NUMERIC(8, 3) >CHECK (weight_kg > 0),
>    stock_quantity  INTEGER NOT >NULL DEFAULT 0 CHECK      (stock_quantity >= 0),
>    created_at      TIMESTAMPTZ >NOT NULL DEFAULT NOW()
>);
>
>CREATE TABLE product_categories (
>    product_id   INTEGER NOT NULL REFERENCES products (product_id)     ON DELETE CASCADE,
>    category_id  INTEGER NOT NULL REFERENCES categories (category_id)   ON DELETE CASCADE,
>    PRIMARY KEY (product_id, category_id)
>);
>
>CREATE TABLE orders (
>    order_id              SERIAL PRIMARY KEY,
>    customer_id           INTEGER NOT NULL REFERENCES customers (customer_id) ON DELETE RESTRICT,
>    order_date            TIMESTAMPTZ NOT NULL DEFAULT NOW(),
>    status                VARCHAR(20) NOT NULL DEFAULT 'pending'
>                          CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
>    shipping_street       VARCHAR(255),
>    shipping_city         VARCHAR(100),
>    shipping_postal_code  VARCHAR(20),
>    shipping_country      VARCHAR(100)
>);
>
>CREATE TABLE order_items (
>   order_id    INTEGER NOT NULL REFERENCES orders (order_id)      ON DELETE CASCADE,
>    product_id  INTEGER NOT NULL REFERENCES products (product_id)  ON DELETE RESTRICT,
>    quantity    INTEGER NOT NULL CHECK (quantity > 0),
>    unit_price  NUMERIC(10, 2) NOT NULL CHECK (unit_price > 0),
>    PRIMARY KEY (order_id, product_id)
>);

>
> ```

> [!NOTE]
> ***Your Answer***
>
> *(Paste written justifications for data types, FK actions, and design decisions here.)*
>
>
>
>

## Exercise 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the relevant theory section.

1. List the seven phases of the database development lifecycle in order. Which phase is this week's focus? *(Section 1)*

> [!NOTE]
> ***Your Answer***
>
> *(1. Requirements Gathering.)*
> *(2. Conceptual Design)*
> *(3. Logical Design <- this week`s focus)*
> *(4. Physical Design)*
> *(5. Implementation)*
> *(6. Testing and Validation)*
> *(7. Maintenance and Evolution)*

2. Explain the transformation rule for mapping a 1:N relationship to the relational model. Why is the foreign key placed on the "many" side? *(Section 3.2)*

> [!NOTE]
> ***Your Answer***
>
> *( Because if you store it on the ONE side, you would need to store multiple IDs, that would violate the automicity. Thats why we store it on the MULTIPLE side for multiple entries.)*
>
>
>
>

3. What is a junction table? When is it needed? Give an example not from TrailShop. *(Section 3.3)*

> [!NOTE]
> ***Your Answer***
>
> *(A junction table is a table created from two table with both of them having MANY attribute? and has a reference of both tables PK are combined into one PK. Its needed to compose a table with relational attributes between the tables that have a MANY attribute. For example we have a item and description table, we make a junction table item_description table with PRIAMRY KEY (item_id, description_id).)*
>
>
>
>

4. When mapping a 1:1 relationship, how do you decide which table gets the foreign key? *(Section 3.4)*

> [!NOTE]
> ***Your Answer***
>
> *(There are four main decision critaria : 1. If one side has mandatory participation and the other optional: Put the FK on the mandatory side so it would always have a value. 2. If both sides are mandatory : Either side works, choose one side that goes naturaly in the queries. 3. if both sides are optional : Put FK on the side that is more likely to have the value. 4.Alternative : Merge both entities into one table if they always exist together.)*
>
>
>
>

5. How does the mapping of a weak entity differ from a strong entity? What happens to the primary key? *(Section 3.5)*

> [!NOTE]
> ***Your Answer***
>
> *(strong entity has a table with its own PK while weak entity has a table with composite PK it includes owners Primary Key as Foreign Key.)*
>
>
>
>

6. Why should you never use `REAL` or `DOUBLE PRECISION` for monetary values? What should you use instead? *(Section 4.1)*

> [!NOTE]
> ***Your Answer***
>
> *(`REAL` and `DOUBLE PRECISION` is usually used for scientific data where the imprecision is OK or for cordinates, it alsi differs in size and for us we need the precision. We should use either an INTEGER or NUMERIC values.)*
>
>
>
>

7. What is the difference between `TIMESTAMP` and `TIMESTAMPTZ`? Which should you prefer and why? *(Section 4.3)*
> [!NOTE]
> ***Your Answer***
>
> *(TIMESTAMP shows only a date and time, while TIMESTAMPTZ shows data and time and also timezone. You should always use TIMESTAMPTZ just so there wouldnt be any timezone bugs when other users are from other time zones.)*
>




8. Explain the difference between `CASCADE` and `RESTRICT` as foreign key delete actions. Give a scenario where each is appropriate. *(Section 6)*
> [!NOTE]
> ***Your Answer***
>
> *(if the parent is being deleted and the child rows are meaningles you should use `CASCADE` but if you dont wanna delete the children you by accident or it has a use you should use `RESTRICT`. For example if the link products are meiningles after the parent deletion u should use `CASCADE`, but if for example there is a customer in order conjuction table and you need to delete the order table you still wanna keep the the user so you should consider a soft delete and use `RESTRICT` a.k.a ON DELETE have a `RESTRICT` and ON UPDATE have a `CASCADE`.)*
>




9. What is an insertion anomaly? Give an example and explain how proper schema design prevents it. *(Section 7)*
> [!NOTE]
> ***Your Answer***
>
> *(Insertion anomaly is when you cannot insert data without inserting other related data with it. For example : You cant add a new category without adding a new product also is from that category. The prevention of anomalies is a schema design NORMALIZATION, it decomposes tables to eliminate redundancy.)*
>




10. What is the difference between a surrogate key and a natural key? Give one advantage of each. *(Section 9)*
> [!NOTE]
> ***Your Answer***
>
> *(A Natural key is a column/s that has a real-world meaning and naturally identifies rach row, while a Surrogated key is an artificialy generated value with no business meaning. You use Surrogated key if the natural one is long, composite or might change and you need consistent JOIN performance OR Natural key where it is short, stable and universally recognized.)*
>




11. Why does PostgreSQL fold unquoted identifiers to lowercase? How does `snake_case` naming help? *(Section 8)*

> [!NOTE]
> ***Your Answer***
>
> *(PostgresSQL automatically make upercase letters into lower case letter unless you quote it , but if you quote it, youll have to use the same naming all around the tables and tht can lead into errors thats why `snake_case` is used so there woulnt be any errors on naming section.)*
>
>
>
>

12. What does `SET NULL` do as a foreign key action? When would you use it instead of `CASCADE`? *(Section 6)*
> [!NOTE]
> ***Your Answer***
>
> *(`SET NULL` makes it so the child would survive after deletion but it would lose its link for example : delete a manager (employee), it would set manager_id to NULL. `SET NULL` is used when a child has some sort of value and the relationship is optional and `CASCADE` when a child is owned by or a component of the parent. )*
>




---

## Exercise 3: Transformation Exercise — Hotel Booking System

### Given ER Diagram

A hotel booking system has the following entities and relationships:

**Entities:**

1. **Hotel** — hotel_id (PK), name, city, star_rating, phone
2. **Room** (weak entity, owned by Hotel) — room_number (partial key), room_type, floor, price_per_night, has_balcony
3. **Guest** — guest_id (PK), first_name, last_name, email, phone, passport_number
4. **Booking** — booking_id (PK), check_in_date, check_out_date, total_amount, status
5. **Service** — service_id (PK), name, description, price (e.g., "Room Service", "Spa", "Airport Shuttle")

**Relationships:**

- Hotel (1) → Room (N): A hotel has many rooms. Each room belongs to exactly one hotel. (Identifying relationship — Room is weak.)
- Guest (1) → Booking (N): A guest can make many bookings. Each booking belongs to one guest.
- Booking (M) ↔ Room (N): A booking can include multiple rooms, and a room can appear in many bookings (over time). The junction records the specific dates.
- Booking (M) ↔ Service (N): A booking can use multiple services, and a service can be used by many bookings. The junction records the date used and quantity.

### Task

1. Write `CREATE TABLE` statements for ALL tables (including junction tables).
2. For each table:
   - Choose appropriate data types
   - Define PK, FK, NOT NULL, UNIQUE, CHECK, and DEFAULT constraints
   - Specify ON DELETE and ON UPDATE actions for all FKs
3. Create the tables in the correct dependency order.
4. Explain why Room is a weak entity and how its PK reflects this.

> [!NOTE]
> ***Your SQL***
>
> ```sql
 CREATE TABLE hotel (
hotel_id INTEGER PRIMARY KEY,
name VARCHAR(100) NOT NULL,
city VARCHAR(100) NOT NULL,
star_rating SMALLINT NOT NULL check(star_rating BETWEEN 1 AND 5),
phone VARCHAR(20) NOT NULL
);



CREATE TABLE room (
hotel_id INTEGER NOT NULL,
room_number INTEGER NOT NULL,
room_type VARCHAR(20) NOT NULL,
floor SMALLINT NOT NULL,
price_per_night NUMERIC(10,2) NOT NULL CHECK(price_per_night >= 0),
has_balcony BOOL NOT NULL DEFAULT FALSE,
PRIMARY KEY (hotel_id, room_number),
FOREIGN KEY (hotel_id) REFERENCES hotel(hotel_id)
ON DELETE CASCADE 
ON UPDATE CASCADE
);

CREATE TABLE guest (
guest_id INTEGER PRIMARY KEY,
first_name VARCHAR(50) NOT NULL,
last_name VARCHAR(50) NOT NULL,
email VARCHAR(254)  UNIQUE NOT NULL,
phone VARCHAR(20) NOT NULL,
passport_number VARCHAR(30) UNIQUE NOT NULL
);

CREATE TABLE booking (
booking_id INTEGER PRIMARY KEY,
guest_id INTEGER NOT NULL,
check_in_date DATE NOT NULL,
check_out_date DATE NOT NULL,
total_amount NUMERIC(10,2) NOT NULL CHECK (total_amount >= 0),
status VARCHAR(20) NOT NULL,
CHECK (check_out_date > check_in_date),
FOREIGN KEY (guest_id) REFERENCES guest(guest_id)
ON DELETE RESTRICT
ON UPDATE CASCADE
);

CREATE TABLE room_booking (
booking_id INTEGER NOT NULL,
hotel_id INTEGER NOT NULL,
room_number INTEGER NOT NULL,
check_in_date DATE NOT NULL,
check_out_date DATE NOT NULL,
CHECK (check_out_date > check_in_date),
PRIMARY KEY (booking_id, hotel_id, room_number),
FOREIGN KEY (booking_id) REFERENCES booking(booking_id)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (hotel_id, room_number) REFERENCES room(hotel_id, room_number)
ON DELETE RESTRICT
ON UPDATE CASCADE
);

CREATE TABLE booking_service(
booking_id INTEGER NOT NULL,
service_id INTEGER NOT NULL,
service_date DATE NOT NULL,
quantity INTEGER NOT NULL CHECK (quantity > 0),
PRIMARY KEY (booking_id, service_id, service_date),
FOREIGN KEY (booking_id) REFERENCES booking(booking_id)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY (service_id) REFERENCES service(service_id)
ON DELETE RESTRICT
ON UPDATE CASCADE

);

CREATE TABLE service (
service_id INTEGER PRIMARY KEY,
name VARCHAR(100) NOT NULL,
description VARCHAR(255),
price NUMERIC(10,2) CHECK(price >= 0) NOT NULL
);
> ```

> [!NOTE]
> ***Your Answer***
>
> *(Room is a weak entity because its own attribute isnt unique and is only a partial key, so we take atributes from hotel_id and room_number to make it unique .)*
>
>
>
>

---

## Exercise 4: Data Type Selection

> [!NOTE]
> ***Your Answers***
> Fill in the **Your Data Type** and **Justification** columns in the table below.
>

For each column described below, choose the best PostgreSQL data type and write a brief justification (1–2 sentences). Do NOT just pick `VARCHAR` or `TEXT` for everything — think carefully about validation, storage, and query needs.

| # | Column Description | Your Data Type | Justification |
|---|---|---|---|
| 1+ | Employee salary (exact, up to €999,999.99) | `NUMERIC(10,2)` | It is usually what you write to in price,salary and everything that sums up the price attributes |
| 2+ | Number of items in stock (never negative, max ~50,000) |`INTEGER + CHECK stock >= 0` | SMALLINT caps out at 36k so INTEGER is bigger and CHECK just doesnt let the number to go bellow 0 |
| 3+ | Whether a user's email is verified | `BOOL` | Boolean just makes it a true or false data so if its verified its gonna be true  |
| 4+ | Customer's date of birth | `DATE` | DATE only posts xxxx.xx.xx way so we dont need any other data type |
| 5+ | Product description (variable length, could be several paragraphs) | `TEXT` | On text theres no limit so it can go pragraphs long if needed |
| 6+ | Country code (always exactly 2 letters, like "FI", "US") | `CHAR(2)` | CAHR(2) is used mostly used for fixed codes |
| 7+ | IP address of a login attempt | `INET` | INET is used to write IPv4 or IPv6 host address data |
| 8+ | Order total (exact, up to €9,999,999.99) | `NUMERIC(10,2)`  | Same as before everything that rounds up around price this data type is used |
| 9+ | GPS latitude of a store location | `DOUBLE PRECISION` | It is used for coordinates because of how many decimal digits it takes |
| 10+ | A unique identifier for API tokens that must be globally unique across distributed systems | `UUID` | Its a universally unique identifier so the globaly unique data fits |
| 11+ | Duration of a video in seconds (always a whole number) | `INTEGER` | In integer there can be a lot of seconds if needed so it doesnt capout like SMALLINT |
| 12+ | Timestamp of when a record was last modified (users in multiple time zones) | `TIMESTAMPTZ` | It shows a date, time and time zones  |
| 13+ | A Finnish phone number like "+358 40 123 4567" | `VARCHAR(20)` | Its a simple number with randomized digits nothing needs to be done more |
| 14+ | A percentage discount (0.00% to 100.00%) | `NUMERIC(5,2)` | Its mostly used for percentage counting from 0.00 to 100.00 |
| 15+ | A product's color options (e.g., a product comes in "red", "blue", "green") | `TEXT[]` | Array with text fits what needs to be done  |

---

## Exercise 5: Constraint Design

For each business rule below, write the appropriate PostgreSQL constraint. Provide the constraint as it would appear inside a `CREATE TABLE` statement or as an `ALTER TABLE` statement.

### Part A: Single-Column Constraints

1. "A product's weight must be greater than zero (if provided)."

2. "Every customer must have an email address."

3. "Product names must be unique."

4. "An employee's hire date defaults to today if not specified."

5. "Order status can only be one of: 'new', 'confirmed', 'shipped', 'delivered', 'returned'."

> [!NOTE]
> ***Your SQL***
>
> ```sql
> 1. product_weight NUMERIC(10,2) CHECK(product_weight > 0),
>
> 2. customer_email VARCHAR(100) NOT NULL,
>
> 3. product_name VARCHAR(100) UNIQUE NOT NULL,
>
> 4. employee_date DATE NOT NULL DEFAULT NOW(),
>
> 5. order_status VARCHAR(20)  NOT NULL CHECK (order_status IN ('new', 'confirmed', 'shipped', 'delivered', 'returned')),
>
>
> ```

### Part B: Multi-Column Constraints

6. "A flight's arrival time must be after its departure time."

7. "In the `enrollments` table, the combination of `student_id` and `course_id` must be unique (a student can only enroll in a course once)."

8. "A discount percentage must be between 0 and 100, inclusive."

> [!NOTE]
> ***Your SQL***
>
> ```sql
>
>  6. ALTER TABLE flights
>     ADD CONSTRAINT check_flight_times CHECK (arrival_time > departure_time);
>
>  7. ALTER TABLE enrollments
>     ADD CONSTRAINT unique_student_course UNIQUE (student_id, course_id);
>
>  8. ALTER TABLE discounts
>     ADD CONSTRAINT check_discount_range CHECK (discount_percentage BETWEEN 0 AND 100);
>
>
> ```

### Part C: Foreign Key Constraints with Actions

9. "When a department is deleted, all employees in that department should have their `department_id` set to NULL (they become unassigned)."

10. "When a customer is deleted, prevent the deletion if the customer has any orders."

11. "When an author is deleted, all their blog posts should be deleted automatically."

12. "When a course is deleted, all enrollments for that course should be removed."

> [!NOTE]
> ***Your SQL***
>
> ```sql
>
>  9.    ALTER TABLE employees
>        ADD FOREIGN KEY (department_id) REFERENCES department(department_id)
>        ON DELETE SET NULL;
>
>  10.   ALTER TABLE orders
>        ADD FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
>        ON UPDATE RESTRICT;
>
>  11.   ALTER TABLE blog_posts
>        ADD FOREIGN KEY (author_id) REFERENCES authors(author_id)
>        ON DELETE CASCADE;
> 
>  12.   ALTER TABLE enrollments
>        ADD FOREIGN KEY (course_id) REFERENCES courses(course_id)
>        ON DELETE CASCADE;
>
> ```

---

## Submission Checklist

- [ ] Exercise 1: `.sql` file with all CREATE TABLE statements + written justifications
- [ ] Exercise 2: All 12 theory review answers
- [ ] Exercise 3: Hotel booking schema with all tables and explanations
- [ ] Exercise 4: Data type selections with justifications for all 15 columns
- [ ] Exercise 5: All 12 constraints written in valid PostgreSQL syntax
