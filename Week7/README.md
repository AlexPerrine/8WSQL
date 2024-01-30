# Week 7 Challenge!

<img src='balance-tree.png' alt="Balance Tree Logo" width=auto height="700">
For more information about the <a href="https://8weeksqlchallenge.com/case-study-7/">week 7</a> challenge. 

### Introduction
The Seventh challenge follows the Balanced Tree Clothing Company which prides themselves on providing an optimised range of clothing and lifestyle wear forthe modern adventurer. The CEO Danny, is asking for assistance for the merchandising team as they analyze their sales performance and to generate basic financial reports to share with the wider business.

### High Level Sales Analysis
1. What was the total quantity sold for all products?
```sql
SELECT 
  products.product_name,
  SUM(sales.qty) AS total_quantity
FROM balanced_tree.sales AS sales
JOIN balanced_tree.product_details AS products
ON sales.prod_id = products.product_id
GROUP BY products.product_name
```
| product\_name                    | total\_quantity |
| -------------------------------- | --------------- |
| White Tee Shirt - Mens           | 3800            |
| Navy Solid Socks - Mens          | 3792            |
| Grey Fashion Jacket - Womens     | 3876            |
| Navy Oversized Jeans - Womens    | 3856            |
| Pink Fluro Polkadot Socks - Mens | 3770            |
| Khaki Suit Jacket - Womens       | 3752            |
| Black Straight Jeans - Womens    | 3786            |
| White Striped Socks - Mens       | 3655            |
| Blue Polo Shirt - Mens           | 3819            |
| Indigo Rain Jacket - Womens      | 3757            |
| Cream Relaxed Jeans - Womens     | 3707            |
| Teal Button Up Shirt - Mens      | 3646            |

2. What is the total generated revenue for all products before discounts?
```sql
SELECT
  SUM(price * qty) AS total_revenue
FROM balanced_tree.sales;
```
| total\_revenue |
| -------------- |
| 1289453        |

3. What was the total discount amount for all products?
```sql
SELECT
    SUM(price * qty - discount)
FROM balanced_tree.sales;
```
| sum     |
| ------- |
| 1106753 |

### Transaction Analysis
1. How many unique transactions were there?
```sql
SELECT
    COUNT(DISTINCT txn_id) AS unique_transactions
FROM balanced_tree.sales;
```
| unique\_transactions |
| -------------------- |
| 2500                 |

2. What is the average unique products purchased in each transaction?
```sql
with transaction_cte AS(
  SELECT
    txn_id,
    COUNT(*) AS product_count
  FROM balanced_tree.sales
  GROUP BY txn_id
)

SELECT
  ROUND(AVG(product_count)) AS average_purchases
FROM transaction_cte
```
| average\_purchases |
| ------------------ |
| 6                  |

3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```sql
with rev_percentile_cte AS(
  SELECT
    txn_id,
    SUM(qty * price) AS revenue
  FROM balanced_tree.sales
  GROUP BY txn_id
)

SELECT
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue)AS pct_25,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue)AS pct_50,
  PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue)AS pct_75
FROM rev_percentile_cte
```
| pct\_25 | pct\_50 | pct\_75 |
| ------- | ------- | ------- |
| 375.75  | 509.5   | 647     |

4. What is the average discount value per transaction?
```sql
with discount_cte AS(
  SELECT
    txn_id,
    SUM(qty * price * discount) AS total_discount
  FROM balanced_tree.sales
  GROUP BY txn_id
)

SELECT
  ROUND(AVG(total_discount)/100, 2) AS average_discount
FROM discount_cte
```
| average\_discount |
| ----------------- |
| 62.49             |
5. What is the percentage split of all transactions for members vs non-members?
```sql
with member_cte AS(
  SELECT
    member,
    COUNT(DISTINCT txn_id) AS total_transaction
  FROM balanced_tree.sales
  GROUP BY member
)

SELECT
  member,
  total_transaction,
  ROUND((total_transaction / SUM(total_transaction) OVER()),2) AS percentage
FROM member_cte
GROUP BY member, total_transaction
```
| member | total\_transaction | percentage |
| ------ | ------------------ | ---------- |
| FALSE  | 995                | 0.40       |
| TRUE   | 1505               | 0.60       |

6. What is the average revenue for member transactions and non-member transactions?
```sql
with member_rev_cte AS(
  SELECT
    member,
    txn_id,
    SUM(qty * price) AS total_rev
  FROM balanced_tree.sales
  GROUP BY member, txn_id
)

SELECT
  member,
  PERCENTILE_CONT(.5) WITHIN GROUP (ORDER BY total_rev) AS average_revenue
FROM member_rev_cte
GROUP BY member
```
| member | average\_revenue |
| ------ | ---------------- |
| FALSE  | 508              |
| TRUE   | 511              |

### Product Analysis
1. What are the top 3 products by total revenue before discount?
```sql
SELECT
  prod.product_name AS product,
  SUM(sales.qty * sales.price) AS total_revenue
FROM balanced_tree.sales AS sales
JOIN balanced_tree.product_details AS prod
ON sales.prod_id = prod.product_id
GROUP BY product
ORDER BY 2 DESC
LIMIT 3
```
| product                      | total\_revenue |
| ---------------------------- | -------------- |
| Blue Polo Shirt - Mens       | 217683         |
| Grey Fashion Jacket - Womens | 209304         |
| White Tee Shirt - Mens       | 152000         |

2. What is the total quantity, revenue and discount for each segment?
```sql
SELECT
  prod.segment_name AS segment,
  SUM(sales.qty) AS quantity,
  SUM(sales.qty * sales.price) AS total_revenue,
  ROUND(
    SUM(sales.qty * sales.price * sales.discount / 100), 2) AS total_discount
FROM balanced_tree.sales AS sales
JOIN balanced_tree.product_details AS prod
ON sales.prod_id = prod.product_id
GROUP BY segment_name
ORDER BY 3 DESC
```
| segment | quantity | total\_revenue | total\_discount |
| ------- | -------- | -------------- | --------------- |
| Shirt   | 11265    | 406143         | 48082.00        |
| Jacket  | 11385    | 366983         | 42451.00        |
| Socks   | 11217    | 307977         | 35280.00        |
| Jeans   | 11349    | 208350         | 23673.00        |

3. What is the top selling product for each segment?
```sql
with rank_final_cte AS(
SELECT
  prod.segment_name AS segment,
  prod.product_name AS product,
  SUM(sales.qty) AS total_quantity,
  RANK() OVER(PARTITION BY prod.segment_name ORDER BY SUM(sales.qty) DESC) AS top_selling
FROM balanced_tree.sales AS sales
JOIN balanced_tree.product_details AS prod
ON sales.prod_id = prod.product_id
GROUP BY 1,2
)

SELECT
  segment,
  product,
  total_quantity
FROM rank_final_cte
WHERE top_selling = 1
```
| segment | product                       | total\_quantity |
| ------- | ----------------------------- | --------------- |
| Jacket  | Grey Fashion Jacket - Womens  | 3876            |
| Jeans   | Navy Oversized Jeans - Womens | 3856            |
| Shirt   | Blue Polo Shirt - Mens        | 3819            |
| Socks   | Navy Solid Socks - Mens       | 3792            |

4. What is the total quantity, revenue and discount for each category?
```sql
select

  prod.category_name                                    as catetory
  , sum(sales.qty)                                      as total_quantity
  , sum(sales.price * sales.qty)                        as total_revenue
  , sum(sales.price * sales.qty * sales.discount) / 100 as total_discount
  
from balanced_tree.sales as sales
inner join balanced_tree.product_details as prod
on sales.prod_id = prod.product_id
group by 1
```
| catetory | total\_quantity | total\_revenue | total\_discount  |
| -------- | --------------- | -------------- | ---------------- |
| Mens     | 22482           | 714120         | 86607.71         |
| Womens   | 22734           | 575333         | 69621.43         |

5. What is the top selling product for each category?
```sql
with top_product_cte as(
select

  prod.category_name                     as category
  , prod.product_name                    as product
  , sum(sales.qty)                       as total_quantity
  , sum(sales.qty * sales.price)       as total_revenue
  , rank() over(partition by prod.category_name
                order by sum(sales.qty)) as product_rank
  
from balanced_tree.sales as sales
inner join balanced_tree.product_details as prod
on sales.prod_id = prod.product_id
group by 1,2
)

select
  category
  , product
  , total_quantity
  , total_revenue
from top_product_cte
where product_rank = 1
```
| category | product                      | total\_quantity | total\_revenue |
| -------- | ---------------------------- | --------------- | -------------- |
| Mens     | Teal Button Up Shirt - Mens  | 3646            | 36460          |
| Womens   | Cream Relaxed Jeans - Womens | 3707            | 37070          |

6. What is the percentage split of revenue by product for each segment?

I felt like adding in more breakdowns to work on the window functions

```sql
with top_segment_cte as(
select

  prod.segment_name              as segment
  , prod.product_name            as product
  , sum(sales.qty * sales.price) as total_revenue

from balanced_tree.sales                 as sales
inner join balanced_tree.product_details as prod
on sales.prod_id = prod.product_id
group by 1,2
)

select
  *
  , sum(total_revenue) over(partition by segment) as segment_total
  , round(100 * total_revenue / sum(total_revenue) 
          over(partition by segment), 2)          as segment_percent
  , sum(total_revenue) over()                     as overall_revenue
  , round(100 * total_revenue / sum(total_revenue) 
          over(), 2)                              as overall_percent
from top_segment_cte
group by 1,2,3
order by segment, total_revenue desc
```
| segment | product                          | total\_revenue | segment\_total | segment\_percent | overall\_revenue | overall\_percent |
| ------- | -------------------------------- | -------------- | -------------- | ---------------- | ---------------- | ---------------- |
| Jacket  | Grey Fashion Jacket - Womens     | 209304         | 366983         | 57.03            | 1289453          | 16.23            |
| Jacket  | Khaki Suit Jacket - Womens       | 86296          | 366983         | 23.51            | 1289453          | 6.69             |
| Jacket  | Indigo Rain Jacket - Womens      | 71383          | 366983         | 19.45            | 1289453          | 5.54             |
| Jeans   | Black Straight Jeans - Womens    | 121152         | 208350         | 58.15            | 1289453          | 9.40             |
| Jeans   | Navy Oversized Jeans - Womens    | 50128          | 208350         | 24.06            | 1289453          | 3.89             |
| Jeans   | Cream Relaxed Jeans - Womens     | 37070          | 208350         | 17.79            | 1289453          | 2.87             |
| Shirt   | Blue Polo Shirt - Mens           | 217683         | 406143         | 53.60            | 1289453          | 16.88            |
| Shirt   | White Tee Shirt - Mens           | 152000         | 406143         | 37.43            | 1289453          | 11.79            |
| Shirt   | Teal Button Up Shirt - Mens      | 36460          | 406143         | 8.98             | 1289453          | 2.83             |
| Socks   | Navy Solid Socks - Mens          | 136512         | 307977         | 44.33            | 1289453          | 10.59            |
| Socks   | Pink Fluro Polkadot Socks - Mens | 109330         | 307977         | 35.50            | 1289453          | 8.48             |
| Socks   | White Striped Socks - Mens       | 62135          | 307977         | 20.18            | 1289453          | 4.82             |

7. What is the percentage split of revenue by segment for each category?
```sql
with top_category_segment_cte as(
select

  prod.category_name             as category
  , prod.segment_name            as segment
  , sum(sales.qty * sales.price) as total_revenue

from balanced_tree.sales                 as sales
inner join balanced_tree.product_details as prod
on sales.prod_id = prod.product_id
group by
    prod.category_id,
    prod.category_name,
    prod.segment_id,
    prod.segment_name
)

select

  category
  , segment
  , total_revenue
  , round(100 * total_revenue / sum(total_revenue) 
          over(partition by category), 2)          as category_segment_percent
          
from top_category_segment_cte
group by 1,2,3
order by category, total_revenue desc
```
| category | segment | total\_revenue | category\_segment\_percent |
| -------- | ------- | -------------- | -------------------------- |
| Mens     | Shirt   | 406143         | 56.87                      |
| Mens     | Socks   | 307977         | 43.13                      |
| Womens   | Jacket  | 366983         | 63.79                      |
| Womens   | Jeans   | 208350         | 36.21                      |

8. What is the percentage split of total revenue by category?
```sql
with top_category_cte as(
select

  prod.category_name             as category
  , sum(sales.qty * sales.price) as total_revenue

from balanced_tree.sales                 as sales
inner join balanced_tree.product_details as prod
on sales.prod_id = prod.product_id
group by
    prod.category_id,
    prod.category_name
)

select

  category
  , total_revenue
  , round(100 * total_revenue / sum(total_revenue) 
          over(), 2)          as category_percent
          
from top_category_cte
group by 1,2
order by category, total_revenue desc
```
| category | total\_revenue | category\_percent |
| -------- | -------------- | ----------------- |
| Mens     | 714120         | 55.38             |
| Womens   | 575333         | 44.62             |

