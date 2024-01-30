# Week 6 Challenge!

<img src='clique-bait.png' alt="Clique Bait Logo" width=auto height="700">
For more information about the <a href="https://8weeksqlchallenge.com/case-study-6/">week 6</a> challenge. 

### Introduction
The Sixth challenge follows the Seafoodprenuer and CEO Danny. I am required to support Danny's vision and analyze the data and come up with creative solutions to calculate funnel rollout rates for Clique Bait's online store

### Digital Analysis Questions
1. How many users are there?
```sql
SELECT
  COUNT(DISTINCT user_id) AS total_users
FROM clique_bait.users
```
| total\_users |
| ------------ |
| 500          |

2. How many cookies does each user have on average?
```sql
with cookie_count_cte AS(
  SELECT 
    user_id,
    COUNT(cookie_id) AS cookie_count
  FROM clique_bait.users
  GROUP BY 1
  ORDER BY 1
)

SELECT
  ROUND(SUM(cookie_count) / COUNT(user_id) ,2) AS average_cookies
FROM cookie_count_cte
```
| average\_cookies |
| ---------------- |
| 3.56             |

3. What is the unique number of visits by all users per month?
```sql
SELECT
  EXTRACT('month'FROM event_time) AS month_number,
  COUNT(DISTINCT visit_id) AS unique_visits
FROM clique_bait.events
GROUP BY 1
ORDER BY 1
```
| month\_number | unique\_visits |
| ------------- | -------------- |
| 1             | 876            |
| 2             | 1488           |
| 3             | 916            |
| 4             | 248            |
| 5             | 36             |

4. What is the number of events for each event type?
```sql
SELECT
  events.event_type,
  identifier.event_name,
  COUNT(events.event_type) AS number_events
FROM clique_bait.events AS events
JOIN clique_bait.event_identifier AS identifier
ON events.event_type = identifier.event_type
GROUP BY 1, 2
ORDER BY 3 DESC
```
| event\_type | event\_name   | number\_events |
| ----------- | ------------- | -------------- |
| 1           | Page View     | 20928          |
| 2           | Add to Cart   | 8451           |
| 3           | Purchase      | 1777           |
| 4           | Ad Impression | 876            |
| 5           | Ad Click      | 702            |

5. What are the top 3 pages by number of views?
```sql
SELECT *
FROM clique_bait.event_identifier
```
| event\_type | event\_name   |
| ----------- | ------------- |
| 1           | Page View     |
| 2           | Add to Cart   |
| 3           | Purchase      |
| 4           | Ad Impression |
| 5           | Ad Click      |

From this query we see that we need to use a WHERE clause to filter on event_type of 1
```sql
SELECT
  pages.page_name,
  COUNT(1) AS page_views
FROM clique_bait.events AS events
JOIN clique_bait.page_hierarchy AS pages
ON events.page_id = pages.page_id
WHERE event_type = 1
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3
```
| page\_name   | page\_views |
| ------------ | ----------- |
| All Products | 3174        |
| Checkout     | 2103        |
| Home Page    | 1782        |

6. What is the number of views and cart adds for each product category?
```sql
SELECT
  pages.product_category,
  SUM(CASE WHEN events.event_type = 1 THEN 1 ELSE 0 END) AS page_views,
  SUM(CASE WHEN events.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds
FROM clique_bait.events AS events
INNER JOIN clique_bait.page_hierarchy AS pages
ON events.page_id = pages.page_id
WHERE pages.product_category IS NOT NULL
GROUP BY 1
```
| product\_category | page\_views | cart\_adds |
| ----------------- | ----------- | ---------- |
| Luxury            | 3032        | 1870       |
| Shellfish         | 6204        | 3792       |
| Fish              | 4633        | 2789       |