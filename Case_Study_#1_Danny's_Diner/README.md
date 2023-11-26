# 🍽️ Case Study #1 - Danny's Diner <!-- omit from toc -->

- [1. Problem Statement](#1-problem-statement)
  - [1.1. What is the total amount each customer spent at the restaurant?](#11-what-is-the-total-amount-each-customer-spent-at-the-restaurant)
  - [1.2. How many days has each customer visited the restaurant?](#12-how-many-days-has-each-customer-visited-the-restaurant)
  - [1.3. What was the first item from the menu purchased by each customer?](#13-what-was-the-first-item-from-the-menu-purchased-by-each-customer)
  - [1.4. What is the most purchased item on the menu and how many times was it purchased by all customers?](#14-what-is-the-most-purchased-item-on-the-menu-and-how-many-times-was-it-purchased-by-all-customers)
  - [1.5. Which item was the most popular for each customer?](#15-which-item-was-the-most-popular-for-each-customer)
  - [1.6. Which item was purchased first by the customer after they became a member?](#16-which-item-was-purchased-first-by-the-customer-after-they-became-a-member)
  - [1.7. Which item was purchased just before the customer became a member?](#17-which-item-was-purchased-just-before-the-customer-became-a-member)
  - [1.8. What is the total items and amount spent for each member before they became a member?](#18-what-is-the-total-items-and-amount-spent-for-each-member-before-they-became-a-member)
  - [1.9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?](#19-if-each-1-spent-equates-to-10-points-and-sushi-has-a-2x-points-multiplier---how-many-points-would-each-customer-have)
  - [1.10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?](#110-in-the-first-week-after-a-customer-joins-the-program-including-their-join-date-they-earn-2x-points-on-all-items-not-just-sushi---how-many-points-do-customer-a-and-b-have-at-the-end-of-january)
- [2. Bonus Questions!](#2-bonus-questions)
  - [2.1. Creating a helper table to track sales with some extra information](#21-creating-a-helper-table-to-track-sales-with-some-extra-information)
  - [2.2. Bonus Question 2 - Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.](#22-bonus-question-2---danny-also-requires-further-information-about-the-ranking-of-customer-products-but-he-purposely-does-not-need-the-ranking-for-non-member-purchases-so-he-expects-null-ranking-values-for-the-records-when-customers-are-not-yet-part-of-the-loyalty-program)


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

----------


#### 1.1. What is the total amount each customer spent at the restaurant?

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

#### 1.2. How many days has each customer visited the restaurant?

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


#### 1.3. What was the first item from the menu purchased by each customer?

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

#### 1.4. What is the most purchased item on the menu and how many times was it purchased by all customers?

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

#### 1.5. Which item was the most popular for each customer?

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

#### 1.6. Which item was purchased first by the customer after they became a member?

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

1. Using `order date` and the `min` function in combination with the `WHERE` clause in the first CTE `first_order` to grab the earliest date after the `join_date`.
2. Then using a little PostgreSQL trick with `DISTINCT ON` which is **PostgresQL specific** and can be used to grab the first record out of each group in a specific order. Read more [here](https://www.geekytidbits.com/postgres-distinct-on/).
3. Finally I join `first_order` to `menu` and we get our answer.

*Note - The answer could depend on how you define when a customer becomes a member. I chose to assume that if there was a sale on the same day that the customer became a member then that sale counts as the first purchase.*

----------

#### 1.7. Which item was purchased just before the customer became a member?

```sql
WITH last_order AS (SELECT
    s.customer_id,
    s.product_id,
    MAX(order_date) as order_date,
    RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date DESC) as rank
FROM dannys_diner.sales as s
RIGHT JOIN dannys_diner.members as m
ON s.customer_id = m.customer_id
WHERE order_date < join_date
GROUP BY s.customer_id, s.product_id, order_date
ORDER BY customer_id, order_date DESC)

SELECT
    customer_id,
    product_name
FROM last_order as l
LEFT JOIN dannys_diner.menu as menu
ON l.product_id = menu.product_id
WHERE rank = 1
ORDER BY customer_id;
```

| customer_id | product_name |
|------------:|-------------:|
|           A |        sushi |
|           A |        curry |
|           B |        sushi |

I took a different approach to this problem compared to the previous, opting to use a `RANK` window function to assign a rank to every sale ordered by date that occured before the `join_date` of the customer. I believe this gave a more accurate answer than just grabbing the first sale as customer `A` purchased 2 items on the same day.

----------

#### 1.8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT
    s.customer_id,
    COUNT(s.customer_id) as items_purchased,
    SUM(price) as total_spent
FROM dannys_diner.sales as s
LEFT JOIN dannys_diner.menu as m
ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members as mem
ON s.customer_id = mem.customer_id
WHERE join_date IS NULL
OR order_date < join_date
GROUP BY s.customer_id
ORDER BY customer_id;
```

| customer_id | items_purchased | total_spent |
|------------:|----------------:|------------:|
|           A |               2 |          25 |
|           B |               3 |          40 |
|           C |               3 |          36 |

To answer this question I used a `JOIN` with a `WHERE` statement specifying that `order_date < join_date`. This results in a table with only sales that were made before the customer became a member, which is then easy to aggregate for the answer.

----------

#### 1.9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT
    customer_id,
    SUM(points) as points
FROM dannys_diner.sales as s
LEFT JOIN (
    SELECT 
        product_id,
        price,
        CASE WHEN product_id = 1 THEN price * 20
        ELSE price * 10
        END as points
    FROM dannys_diner.menu
    ) as points_menu
ON s.product_id = points_menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | points | total_spent |
|------------:|-------:|------------:|
|           A |    860 |          25 |
|           B |    940 |          40 |
|           C |    360 |          36 |

This problem can be solved using the trusty `CASE` statement. Since we only have 1 product that has a different price, we can define that as our first case, and then use `ELSE` for all the remaining products. 

It might be worth noting that for a real restaurant with many different products, adding a product multiplier to the `menu` table may be an easier and more scalable solution than hard-coding a `CASE` statement, as products can come and go on the menu.

----------

#### 1.10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
WITH points_menu AS (
    SELECT 
        product_id,
        price,
        CASE WHEN product_id = 1 THEN price * 20
        ELSE price * 10
        END as points
    FROM dannys_diner.menu),

jan_adjusted_points AS (SELECT 
    s.customer_id,
    join_date,
    order_date,
    s.product_id,
    pm.points,
    CASE WHEN order_date BETWEEN join_date AND join_date + INTERVAL '7 day' THEN points * 2
    ELSE points END as member_points
FROM dannys_diner.sales as s
LEFT JOIN points_menu as pm
ON s.product_id = pm.product_id
LEFT JOIN dannys_diner.members as mem
ON s.customer_id = mem.customer_id
WHERE s.customer_id IN (SELECT customer_id FROM dannys_diner.members)
AND order_date < '2021-02-01'
ORDER BY s.customer_id, order_date)

SELECT
    customer_id,
    SUM(member_points)
FROM jan_adjusted_points
GROUP BY customer_id;
```

| customer_id |  sum |
|------------:|-----:|
|           A | 1370 |
|           B | 1140 |

1. Using the point mentioned in the previous solution I solved this question by creating a column called `points` from the `menu` table
2. Then multiplying the points by using a `CASE` statement to check whether the date lies between the `join_date` and `join_date + INTERVAL '7 day'` lets us define the 1 week window where points are worth double
3. Finally a `WHERE` statement which only accepts customers if they are in the `members` table `AND` the `order_date < '2021-02-01'`

P.S. US formatted dates are a nightmare 🤢 and I sincerely hope one day we all use the same format...

----------

### 2. Bonus Questions!

#### 2.1. Creating a helper table to track sales with some extra information

```sql
SELECT
    s.customer_id,
    s.order_date,
    menu.product_name,
    menu.price,
    CASE WHEN s.order_date >= members.join_date THEN 'Y'
    ELSE 'N' END AS member
FROM dannys_diner.sales as s
LEFT JOIN dannys_diner.menu as menu
ON s.product_id = menu.product_id
LEFT JOIN dannys_diner.members as members
ON s.customer_id = members.customer_id
ORDER BY s.customer_id, s.order_date
```

| customer_id | order_date | product_name | price | member |
|------------:|-----------:|-------------:|------:|-------:|
|           A | 2021-01-01 |        sushi |    10 |      N |
|           A | 2021-01-01 |        curry |    15 |      N |
|           A | 2021-01-07 |        curry |    15 |      Y |
|           A | 2021-01-10 |        ramen |    12 |      Y |
|           A | 2021-01-11 |        ramen |    12 |      Y |
|           A | 2021-01-11 |        ramen |    12 |      Y |
|           B | 2021-01-01 |        curry |    15 |      N |
|           B | 2021-01-02 |        curry |    15 |      N |
|           B | 2021-01-04 |        sushi |    10 |      N |
|           B | 2021-01-11 |        sushi |    10 |      Y |
|           B | 2021-01-16 |        ramen |    12 |      Y |
|           B | 2021-02-01 |        ramen |    12 |      Y |
|           C | 2021-01-01 |        ramen |    12 |      N |
|           C | 2021-01-01 |        ramen |    12 |      N |
|           C | 2021-01-07 |        ramen |    12 |      N |

Another nice little `CASE` statement to help Danny out.

----------

#### 2.2. Bonus Question 2 - Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

```sql
WITH sales_full AS (SELECT
    s.customer_id,
    s.order_date,
    menu.product_name,
    menu.price,
    CASE WHEN s.order_date >= members.join_date THEN 'Y'
    ELSE 'N' END AS member
FROM dannys_diner.sales as s
LEFT JOIN dannys_diner.menu as menu
ON s.product_id = menu.product_id
LEFT JOIN dannys_diner.members as members
ON s.customer_id = members.customer_id
ORDER BY s.customer_id, s.order_date)

SELECT
    *,
    CASE WHEN member = 'N' THEN NULL
    ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
    END AS ranking
FROM sales_full
```

| customer_id | order_date | product_name | price | member | ranking |
|------------:|-----------:|-------------:|------:|-------:|--------:|
|           A | 2021-01-01 |        sushi |    10 |      N |    None |
|           A | 2021-01-01 |        curry |    15 |      N |    None |
|           A | 2021-01-07 |        curry |    15 |      Y |       1 |
|           A | 2021-01-10 |        ramen |    12 |      Y |       2 |
|           A | 2021-01-11 |        ramen |    12 |      Y |       3 |
|           A | 2021-01-11 |        ramen |    12 |      Y |       3 |
|           B | 2021-01-01 |        curry |    15 |      N |    None |
|           B | 2021-01-02 |        curry |    15 |      N |    None |
|           B | 2021-01-04 |        sushi |    10 |      N |    None |
|           B | 2021-01-11 |        sushi |    10 |      Y |       1 |
|           B | 2021-01-16 |        ramen |    12 |      Y |       2 |
|           B | 2021-02-01 |        ramen |    12 |      Y |       3 |
|           C | 2021-01-01 |        ramen |    12 |      N |    None |
|           C | 2021-01-01 |        ramen |    12 |      N |    None |
|           C | 2021-01-07 |        ramen |    12 |      N |    None |

Using the previous query, we can simply add a `RANK` window function to be able to keep track of the rank of each product per customer!

----------

Wow, you made it to the end! [Click Here](https://github.com/NeyasG/8-Week-SQL-Challenge/tree/main/Case%20Study%20%232%20-%20Pizza%20Runner) for the next case study.