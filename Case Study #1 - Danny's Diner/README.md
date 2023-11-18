# 🍽️ Case Study #1 - Danny's Diner <!-- omit from toc -->

- [1. Problem Statement](#1-problem-statement)
- [2. Questions:](#2-questions)
  - [2.1. What is the total amount each customer spent at the restaurant?](#21-what-is-the-total-amount-each-customer-spent-at-the-restaurant)
  - [2.2. How many days has each customer visited the restaurant?](#22-how-many-days-has-each-customer-visited-the-restaurant)
  - [2.3. What was the first item from the menu purchased by each customer?](#23-what-was-the-first-item-from-the-menu-purchased-by-each-customer)
  - [2.4. What is the most purchased item on the menu and how many times was it purchased by all customers?](#24-what-is-the-most-purchased-item-on-the-menu-and-how-many-times-was-it-purchased-by-all-customers)
  - [2.5. Which item was the most popular for each customer?](#25-which-item-was-the-most-popular-for-each-customer)
  - [2.6. Which item was purchased first by the customer after they became a member?](#26-which-item-was-purchased-first-by-the-customer-after-they-became-a-member)


### 1. Problem Statement

We're here to help Danny with his restaurant by providing insights using some data he has gathered.

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

Danny has shared with us 3 key datasets for this case study:

Table 1: `sales`

| customer_id | order_date | product_id |
| :---------: | :--------: | :--------: |
|      A      | 2021-01-01 |     1      |
|      A      | 2021-01-01 |     2      |
|      A      | 2021-01-07 |     2      |
|      A      | 2021-01-10 |     3      |
|      A      | 2021-01-11 |     3      |
|      A      | 2021-01-11 |     3      |
|      B      | 2021-01-01 |     2      |
|      B      | 2021-01-02 |     2      |
|      B      | 2021-01-04 |     1      |
|      B      | 2021-01-11 |     1      |
|      B      | 2021-01-16 |     3      |
|      B      | 2021-02-01 |     3      |
|      C      | 2021-01-01 |     3      |
|      C      | 2021-01-01 |     3      |
|      C      | 2021-01-07 |     3      |

Table 2: `menu`

| product_id | product_name | price |
| :--------: | :----------: | :---: |
|     1      |    sushi     |  10   |
|     2      |    curry     |  15   |
|     3      |    ramen     |  12   |

Table 3: `members`

| customer_id | join_date  |
| :---------: | :--------: |
|      A      | 2021-01-07 |
|      B      | 2021-01-09 |


[See the original here](https://8weeksqlchallenge.com/case-study-1/)

----

### 2. Questions:
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

----------


#### 2.1. What is the total amount each customer spent at the restaurant?

```sql
SELECT 
    customer_id, 
    SUM(price) as price
FROM dannys_diner.sales as s
LEFT JOIN dannys_diner.menu as m
ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | price |
|------------:|------:|
|           A |    76 |
|           B |    74 |
|           C |    36 |

Customer A spend the most with £76 spent!

----------

#### 2.2. How many days has each customer visited the restaurant?

```sql
SELECT 
    customer_id, 
    COUNT(DISTINCT order_date)
FROM dannys_diner.sales
GROUP BY customer_id;
```

| customer_id | count |
|------------:|------:|
|           A |     4 |
|           B |     6 |
|           C |     2 |

Another straightforward query, with customer B having the most visits at 6!

----------


#### 2.3. What was the first item from the menu purchased by each customer?

```sql
WITH sales_ranked AS (SELECT
    customer_id,
    order_date,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) as rank,
    product_id
FROM dannys_diner.sales),

first_purchase AS (SELECT
    DISTINCT customer_id,
    product_id
FROM sales_ranked
WHERE rank = 1)

SELECT 
    customer_id,
    product_name
FROM first_purchase as f
LEFT JOIN dannys_diner.menu as m
ON f.product_id = m.product_id
ORDER BY customer_id;
```

| customer_id | product_name |
|------------:|-------------:|
|           A |        sushi |
|           A |        curry |
|           B |        curry |
|           C |        ramen |

Alright now the fun questions begin! 

Not only are there many ways to get the first purchase, but customer `A` bought 2 items on their first visit to the restaurant. Therefore it may be wise to grab both from the table, hence using the `RANK` function.

Steps:

1. We create a table from `sales` and use a window function to rank the orders by date per customer
2. Then I created another CTE to only grab the top ranked sales
3. Finally I joined the `menu` table to grab the `product_name`.

This likely could be improved by reducing the amount of CTE's but for now it's a straightforward solution.

Actually it doesn't matter here whether we use `RANK` or `DENSE_RANK` because we're only interested in the first entry, or `RANK = 1`. However if we were dealing with anything past that we'd need to beware of the different behaviours of `RANK` and `DENSE_RANK`!

----------

#### 2.4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT
    m.product_name,
    COUNT(s.product_id) as orders
FROM dannys_diner.sales as s
LEFT JOIN dannys_diner.menu as m
ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY orders DESC
LIMIT 1;
```

| product_name | orders |
|-------------:|-------:|
|        ramen |      8 |

Another fairly straightforward solution. 

1. Group by `product_name` (after merging `sales` with `menu`) and count the orders
2. Then order by `order DESC`
3. Limit to the top result `LIMIT 1`

It seems 🍜 `ramen` was the winner!

----------

#### 2.5. Which item was the most popular for each customer?

```sql
WITH ranked_sales AS (SELECT 
    customer_id,
    product_id,
    COUNT(product_id) as orders,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(customer_id) DESC) as rank
FROM dannys_diner.sales
GROUP BY customer_id, product_id
ORDER BY customer_id, rank)

SELECT
    r.customer_id,
    product_name,
    orders
FROM ranked_sales as r
LEFT JOIN dannys_diner.menu as m
ON r.product_id = m.product_id
WHERE rank = 1
ORDER BY customer_id;
```

| customer_id | product_name | orders |
|------------:|-------------:|-------:|
|           A |        ramen |      3 |
|           B |        sushi |      2 |
|           B |        curry |      2 |
|           B |        ramen |      2 |
|           C |        ramen |      3 |

I broke this problem down into 2 steps:

1. Rank the items per customer
2. Grab the highest ranked item for each customer

Using the first CTE I not only use a window function to rank items by number of order per customer but I also need to track how many orders each customer has in a seperate column to display in the final table. Hence `COUNT(product_id) as orders`. Since we're grouping by customer_id and product_id this leaves only the last table to filter for the most popular items using our rank column and then grab the `product_name` from `menu`.

Looks like customer `B` has a very diverse taste!

----------

#### 2.6. Which item was purchased first by the customer after they became a member?

```sql
WITH first_order AS (SELECT
    DISTINCT ON (s.customer_id) s.customer_id,
    s.product_id,
    MIN(order_date) as order_date
FROM dannys_diner.sales as s
RIGHT JOIN dannys_diner.members as m
ON s.customer_id = m.customer_id
WHERE order_date >= join_date
GROUP BY s.customer_id, s.product_id
ORDER BY customer_id, order_date ASC)

SELECT
    customer_id,
    product_name
FROM first_order as f
LEFT JOIN dannys_diner.menu as menu
ON f.product_id = menu.product_id
ORDER BY customer_id;
```

| customer_id | product_name |
|------------:|-------------:|
|           A |        curry |
|           B |        sushi |

A very interesting question, that I broke down like so:

1. 
