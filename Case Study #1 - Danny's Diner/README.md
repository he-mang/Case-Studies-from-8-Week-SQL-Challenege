# 🍜 Case Study #1: Danny's Diner 
<img width="500" height="520" alt="1" src="https://github.com/user-attachments/assets/f4cb13e8-38b6-43a8-b5de-7217b0e37362" />

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

This case study is based on information provided in the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Business Task
Danny wants to analyze his customers’ visiting patterns, spending habits, and favorite menu items to provide a more personalized experience. He plans to use these insights to evaluate expanding the loyalty program and needs basic datasets for his team to inspect without using SQL.

***

## Entity Relationship Diagram
<img width="500" height="520" alt="image" src="https://github.com/user-attachments/assets/b40fbd8e-03fb-485c-a446-c80e95f27ca1" />

***

## Question and Solution

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT 
	s.customer_id,
    SUM(m.price) AS total_amount
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY customer_id;
````

#### Explanation:
- **JOIN** the `sales` and `menu` tables on `product_id` to link each purchase to its price.
- **SUM** the `price` values to calculate the total amount spent by each customer.
- **GROUP BY** `customer_id` to show total spending for each individual customer.

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT 
	customer_id, 
    COUNT(DISTINCT order_date) AS no_of_days_visited
FROM sales
GROUP BY customer_id;
````

#### Explanation:
- **COUNT(DISTINCT)** counts unique `order_date` entries for each customer.
- **DISTINCT** avoids double-counting days—for example, two visits on 2021-01-07 count as 1 day.
- **GROUP BY** `customer_id` aggregates the results per customer to show their total visit days.

#### Answer:
| customer_id | no_of_days_visited |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

***

**3. What was the first item from the menu purchased by each customer??**

````sql
WITH cte1 AS (
	SELECT 
		s.customer_id,
		s.order_date,
		m.product_name,
		DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS drnk
	FROM sales s
	JOIN menu m
		ON s.product_id	 =  m.product_id
)
SELECT
	customer_id,
    product_name
FROM cte1
WHERE drnk = 1
GROUP BY customer_id, product_name;
````

#### Explanation:
- Use a **CTE** (`cte1`) to join `sales` and `menu` on `product_id` to get each customer’s purchases with product names.
- Apply **DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date)** to rank each customer’s orders by date.
- Filter for **drnk = 1** to get only the first purchase(s) of each customer.
- GROUP BY `customer_id`, `product_name` ensures each customer’s first purchased product(s) appear once.

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | sushi        | 
| A           | curry        | 
| B           | curry        | 
| C           | ramen        |

****

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT
	m.product_name,
	COUNT(s.product_id) AS sales_count
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY sales_count DESC
LIMIT 1;
````

#### Explanation:
- **JOIN** `sales` and `menu` on `product_id` to link each sale to its product name.
- **COUNT** the number of times each product was sold to calculate total sales per item.
- **GROUP BY** `product_name` aggregates the counts per product.
- **ORDER BY sales_count DESC** sorts items from most to least sold.
- **LIMIT 1** returns only the single most popular menu item.

#### Answer:
| product_name | sales_count | 
| ----------- | ----------- |
| ramen | 8 |

***

**5. Which item was the most popular for each customer?**

````sql
WITH cte2 AS (
	SELECT
		s.customer_id,
		m.product_name,
		COUNT(s.product_id) AS purchase_count,
		DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS drnk
	FROM sales s
	JOIN menu m
		ON s.product_id = m.product_id
	GROUP BY s.customer_id, m.product_name
	ORDER BY purchase_count DESC
)
SELECT 
	customer_id,
    product_name,
    purchase_count
FROM cte2
WHERE drnk = 1;
````

#### Explanation:
- Use a **CTE** (`cte2`) to calculate the number of times each customer purchased each product.
- Apply **DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_id) DESC)** to rank products per customer based on purchase frequency.
- **WHERE drnk = 1** filters to only include the most frequently purchased product(s) for each customer.
- The final **SELECT** shows `customer_id`, their top `product_name`, and how many times they purchased it.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C prefer ramen, while Customer B enjoys all the food items on the menu.

***

**6. Which item was purchased first by the customer after they became a member?**

````sql
WITH cte3 AS(
	SELECT 
		s.customer_id, s.order_date,
		m.product_name,
		mb.join_date,
		ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rown
	FROM sales s
	JOIN menu m
		ON s.product_id = m.product_id
	JOIN members mb
		ON s.customer_id = mb.customer_id
	WHERE s.order_date > mb.join_date
)
SELECT customer_id, product_name
FROM cte3
WHERE rown = 1;
````

#### Explanation:
- Use a **CTE** (`cte3`) to join `sale`s, `menu`, and `members` tables, linking purchases to product names and membership join dates.
- Apply **ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date)** to rank each customer’s purchases after joining.
- **WHERE s.order_date > join_date** ensures only purchases made after becoming a member are considered.
- **WHERE rown = 1** selects the very first purchase each customer made after joining.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

***

**7. Which item was purchased just before the customer became a member?**

````sql
WITH cte4 AS(
	SELECT 
		s.customer_id, s.order_date,
		m.product_name,
		mb.join_date,
		ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rown
	FROM sales s
	JOIN menu m
		ON s.product_id = m.product_id
	JOIN members mb
		ON s.customer_id = mb.customer_id
	WHERE s.order_date < mb.join_date
)
SELECT customer_id, product_name
FROM cte4
WHERE rown = 1;
````

#### Explanation:
- Use a **CTE** (`cte4`) to join `sales`, `menu`, and `members` tables to link purchases with product names and membership join dates.
- Apply **ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date DESC)** to rank purchases before membership, with the most recent first.
- **WHERE s.order_date < join_date** ensures only purchases made before joining are considered.
- **WHERE rown = 1** selects the last item each customer bought just before becoming a member.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| B           | sushi        |

***

**8. What is the total items and amount spent for each member before they became a member?**

````sql
SELECT
	s.customer_id, 
    COUNT(s.product_id) AS total_items,
    SUM(m.price) AS total_sales
FROM sales s
JOIN members mb
	ON s.customer_id = mb.customer_id
    AND s.order_date < mb.join_date
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
````

#### Explanation:
- **JOIN** `sales` with `members` to focus on purchases made by customers before their membership (`s.order_date < join_date`).
- **JOIN** with `menu` to access product prices.
- **COUNT(product_id)** calculates the total number of items each customer bought before joining.
- **SUM(price)** calculates the total amount spent by each customer before membership.
- **GROUP BY customer_id** aggregates results per customer, and **ORDER BY customer_id** sorts the output for clarity.

#### Answer:
| customer_id | total_items | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

````sql
WITH cte5 AS(
	SELECT 
		product_id,
		CASE
			WHEN product_id = 1 THEN price * 20
			ELSE price * 10
		END AS points
	FROM menu
)
SELECT
	s.customer_id,
    SUM(cte5.points) AS total_points
FROM sales s
JOIN cte5
	ON s.product_id = cte5.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
````

#### Explanation:
- Use a **CTE** (`cte5`) to calculate points for each product:
	- Regular items: 10 points per $1 spent (`price * 10`).
	- Sushi (product_id = 1): 2× points multiplier, so 20 points per $1 spent (`price * 20`).
- **JOIN** sales with the CTE to assign points for each customer purchase.
- **SUM(points)** calculates total points per customer.
- **GROUP BY customer_id** aggregates points for each customer, and **ORDER BY customer_id** sorts the results.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**

````sql
WITH cte6 AS(
	SELECT
		customer_id,
		join_date,
		DATE_ADD(join_date, INTERVAL 6 DAY) AS  valid_date,
		LAST_DAY('2021-01-31') AS last_date
	FROM members
)
SELECT
	s.customer_id,
    SUM(
		CASE
			WHEN m.product_id = 1 THEN m.price * 20
            WHEN s.order_date BETWEEN cte6.join_date AND cte6.valid_date THEN m.price * 20
            ELSE m.price * 10
		END) AS points
FROM sales s
JOIN cte6
	ON s.customer_id = cte6.customer_id
    AND s.order_date BETWEEN cte6.join_date AND cte6.last_date
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
````

#### Explanation:
- Use a CTE (cte6) to calculate:
	- Each customer’s join_date.
	- Their 7-day bonus window (join_date + 6 days).
	- The last date of January (2021-01-31).
- Apply a CASE expression to assign points:
	- Sushi (product_id = 1): earns 20 points per $1 spent (2× multiplier).
	- Within first week after joining: all items earn 20 points per $1.
	- Otherwise: items earn 10 points per $1.
- Restrict purchases to the period between join_date and January 31.
- SUM(points) gives total points per customer, grouped by customer_id.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1020 |
| B           | 320 |


























































































