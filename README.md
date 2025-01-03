# Danny's Diner Case Study

## Project Overview
Danny's Diner is a restaurant case study designed to explore SQL querying techniques and analyze customer behavior using structured datasets. This project involves three main tables: `sales`, `menu`, and `members`. Each table provides valuable insights into customer spending habits, menu preferences, and membership activity. Using SQL, the project answers various analytical questions related to customer behavior, sales performance, and membership benefits.

## Dataset Description

### Tables:
1. **sales**:
   - Contains transaction details for each customer.
   - **Columns:**
     - `customer_id`: ID of the customer.
     - `order_date`: Date of the transaction.
     - `product_id`: ID of the purchased product.

2. **menu**:
   - Describes the menu items available at the diner.
   - **Columns:**
     - `product_id`: ID of the product.
     - `product_name`: Name of the product.
     - `price`: Price of the product.

3. **members**:
   - Tracks customer membership information.
   - **Columns:**
     - `customer_id`: ID of the customer.
     - `join_date`: Date the customer joined the membership program.

## Questions and SQL Solutions

### 1. Total Spending per Customer
Query to calculate the total amount each customer spent at the diner.
```sql
SELECT s.customer_id, SUM(m.price) as total_amount_each_customer_spent
FROM sales as s
JOIN menu as m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

### 2. Number of Days Visited per Customer
Query to count the number of distinct days each customer visited the diner.
```sql
SELECT customer_id, COUNT(DISTINCT order_date) as total_visits
FROM sales
GROUP BY customer_id;
```

### 3. First Menu Item Purchased per Customer
Query to determine the first item purchased by each customer.
```sql
SELECT customer_id, order_date, product_name AS first_item
FROM (
  SELECT s.customer_id, s.order_date, m.product_name,
         ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS row_num
  FROM sales as s
  JOIN menu as m ON s.product_id = m.product_id
) subquery
WHERE row_num = 1;
```

### 4. Most Purchased Item and Frequency
Query to find the most purchased menu item and its total purchases.
```sql
SELECT TOP 1 m.product_name, COUNT(*) as purchase_count
FROM sales as s
JOIN menu as m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY purchase_count DESC;
```

### 5. Most Popular Item per Customer
Query to identify the most popular item for each customer.
```sql
SELECT customer_id, product_name, total_orders
FROM (
  SELECT s.customer_id, m.product_name, COUNT(*) AS total_orders,
         ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS rn
  FROM sales as s
  JOIN menu as m ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
) subquery
WHERE rn = 1;
```

### 6. First Item Purchased After Membership
Query to find the first item purchased by a customer after joining the membership program.
```sql
SELECT t.customer_id, t.first_purchase_date, m.product_name
FROM (
  SELECT s.customer_id, MIN(s.order_date) AS first_purchase_date
  FROM sales as s
  JOIN members as mm ON s.customer_id = mm.customer_id
  WHERE s.order_date > mm.join_date
  GROUP BY s.customer_id
) as t
JOIN sales AS s ON t.customer_id = s.customer_id AND t.first_purchase_date = s.order_date
JOIN menu AS m ON s.product_id = m.product_id;
```

### 7. Item Purchased Before Membership
Query to find the last item purchased by a customer before joining the membership program.
```sql
SELECT customer_id, product_name
FROM (
  SELECT s.customer_id, m.product_name, MAX(s.order_date) AS last_order
  FROM sales as s
  JOIN members as mm ON s.customer_id = mm.customer_id
  JOIN menu as m ON s.product_id = m.product_id
  WHERE s.order_date < mm.join_date
  GROUP BY s.customer_id, m.product_name
) subquery;
```

### 8. Spending Before Membership
Query to calculate the total items and amount spent by each customer before joining the membership program.
```sql
SELECT s.customer_id, COUNT(s.product_id) AS total_items, SUM(m.price) AS total_amount_spent
FROM sales AS s
JOIN members AS mm ON s.customer_id = mm.customer_id
JOIN menu AS m ON s.product_id = m.product_id
WHERE s.order_date < mm.join_date
GROUP BY s.customer_id;
```

### 9. Points Calculation
Query to calculate points for each customer with a special multiplier for sushi.
```sql
SELECT s.customer_id,
       SUM(CASE WHEN m.product_name = 'sushi' THEN 2 * m.price ELSE m.price END) * 10 AS total_points
FROM sales AS s
JOIN menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

### 10. Points Calculation in the First Week of Membership
Query to calculate points in the first week after membership.
```sql
SELECT sales.customer_id,
       SUM(CASE WHEN order_date <= DATEADD(DAY, 6, join_date) THEN menu.price * 2 ELSE menu.price END) * 10 AS total_points
FROM sales
JOIN members ON sales.customer_id = members.customer_id
JOIN menu ON sales.product_id = menu.product_id
WHERE YEAR(order_date) = 2021 AND MONTH(order_date) = 1
GROUP BY sales.customer_id;
```

## Tools and Technologies
- SQL Database Management System (DBMS)
- Structured Query Language (SQL)
- Relational Data Modeling

## Key Insights
- Customer spending and visitation patterns provide critical insights into their dining habits.
- Membership programs influence purchasing behavior, with notable changes in spending after joining.
- Menu item popularity varies significantly across customers and impacts overall sales.

## How to Use
1. Set up a SQL database and execute the provided `CREATE` and `INSERT` statements to create and populate the tables.
2. Use the queries in the corresponding sections to analyze the data.
3. Modify or extend the queries for additional insights as needed.

Enjoy exploring Danny's Diner data!

