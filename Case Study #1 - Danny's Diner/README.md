# Danny's Diner Sales & Loyalty Program Analysis

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Insights](#insights)
- [Question and Solution](#question-and-solution)

This case study is based on information provided in the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Project Goal: Unlocking Customer Value
Danny's Diner, a Japanese food restaurant, is seeking to understand its early operational data (January 2021) to optimize its customer experience and evaluate the effectiveness of its pilot loyalty program. This project uses SQL to transform raw sales and membership data into actionable business insights.

***

## Entity Relationship Diagram
<img width="500" height="520" alt="image" src="https://github.com/user-attachments/assets/b40fbd8e-03fb-485c-a446-c80e95f27ca1" />

- **Sales:** The `sales` table captures all `customer_id` level purchases with an corresponding `order_date` and `product_id` information for when and what menu items were ordered.

- **Menu:** The `menu` table maps the `product_id` to the actual `product_name` and `price` of each menu item.

- **Members:** The final `members` table captures the `join_date` when a `customer_id` joined the beta version of the Danny’s Diner loyalty program.

***

## Key Business Insights & Recommendations
The analysis identified distinct customer patterns and opportunities to refine the loyalty program.

### 1. Customer Engagement & Spending Habits
| Metric			| Customer A  			| Customer B 												| Customer C 						| Overall Trend																|
| ----------------- | --------------------- | --------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------- |
| Total Spend		| $76					| $74														| $36								| **Customer A** is the highest spendor.									|
| Visit Frequency	| 4						| 6															| 2									| **Customer B** is the most frequent visitor.								|
| Favorite Item(s)	| **Ramen** (3 times)	| **Sushi, Curry, Ramen** (2 times each - _diverse_)		| **Ramen** (3 times)				| **Ramen** is the clear favorite for Customers A and C.					|

- **Insight:** The highest spender (A) is not the most frequent visitor (B). This suggests Customer A responds well to a higher Average Order Value (AOV), while Customer B represents a high-volume loyalty target.
- **Recommendation:** Implement a **"Frequency Reward"** specifically for high-frequency visitors like Customer B (e.g., a free side dish after 5 visits) to drive volume.

### 2. Loyalty Program Evaluation
The data provides compelling evidence on customer behavior _just_ before and _just_ after joining the loyalty program.
- **Pre-Membership Behavior**
  - **Customer A** spent $25 (2 items) before joining.
  - **Customer B** spent $40 (3 items) before joining.
  - The **last item purchased before joining for both A and B was Sushi.**
- **Post-Membership Behavior:**
  - **Customer A's** first purchase was **Ramen.**
  - **Customer B's** first purchase was **Sushi.**
- **Insight:** Both customers' **"last non-member purchaase** was Sushi, and for Customer B, the **"first member purchase** was also Sushi. This suggests the high-value Sushi item might be a gateway product to membership or an item they were willing to try before committing to the program.
- **Recommendation:** Consider **offering a high-value incentive** (e.g., 50% off a premium item like Sushi) to customers who are **one purchase away** from the loyalty program entry criteria.

### 3. The Power of Ramen
- **Global Popularity: Ramen** is the undisputed most purchased item on the entire menu (8 total sales).
- **Insight:** Ramen is the diner's signature dish and biggest traffic driver.
- **Recommendation:** Use the popularity of Ramen as a lever for the loyalty program. Exclude it from non-member discounts, but offer a **"Double Points Day"** on Ramen for members only to reinforce loyalty value.

## Points Program Impact
The complex points calculation reveals how the new bonus structures affect customer standing.
| Customer		| Total Points (Standard Rules) | Total Points (Jan 2021 w/ 7-Day Bonus) 	| Impact 																			|
| ------------- | ----------------------------- | ----------------------------------------- | --------------------------------------------------------------------------------- | 
| Customer A 	| 860							| 1020										| 160 extra points due to bonus window..											|
| Customer B 	| 940							| 320										| 40 points lost due to only purchases _after_ join date being countedd for bonus. 	|
| Customer C 	| 360							| 0 (Non-member) 							| N/A																				| 

-**Insight:** The calculation shows Customer A's points dramatically increased, while Customer B's points decreased significantly. **This is a critical finding!** The problem is how the points system was defined: standard rules apply to all sales, but the bonus rule only applies to sales **after** joining. 




---

- Customer B is the most frequent visitor with 6 visits in Jan 2021.
- Danny’s Diner’s most popular item is ramen.
- Customer A and C loves ramen whereas Customer B seems to enjoy sushi, curry and ramen equally.
- Customer A is the 1st member of Danny’s Diner and his first order is ramen.
- The last item ordered by Customers A and B before they became members are sushi. Does it mean that ramen is the deciding factor? It must be really delicious for them to sign up as members!
- Before they became members, both Customers A and B spent $25 and $40.
- Throughout Jan 2021, their points for Customer A: 860, Customer B: 940 and Customer C: 360.

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

***

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
- Use a **CTE** (`cte6`) to calculate:
	- Each customer’s `join_dat`e.
	- Their 7-day bonus window (`join_date` + 6 days).
	- The last date of January (`2021-01-31`).
- Apply a **CASE** expression to assign points:
	- **Sushi (product_id = 1):** earns 20 points per $1 spent (2× multiplier).
	- **Within first week after joining:** all items earn 20 points per $1.
	- **Otherwise:** items earn 10 points per $1.
- Restrict purchases to the period **between join_date and January 31**.
- **SUM(points)** gives total points per customer, grouped by `customer_id`.

#### Assumptions:
- Before membership (up to the join date), each $1 spent earns **10 points**, except sushi which earns **20 points per $1**.
- From **Day 1 to Day 7 of membership**, every item earns **20 points per $1**.
- From **Day 8 to January 31**, regular items return to **10 points per $1**, while sushi continues to earn **20 points per $1**.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1020 |
| B           | 320 |

***

## BONUS QUESTIONS

**Join All The Things**

**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**

````sql
SELECT
	s.customer_id, s.order_date,
    m.product_name, m.price,
    CASE
		WHEN s.order_date < mb.join_date THEN 'N'
        WHEN s.order_date >= mb.join_date THEN 'Y'
        ELSE 'N'
	END AS member_status
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
LEFT JOIN members mb
	ON s.customer_id = mb.customer_id
ORDER BY s.customer_id, s.order_date;
````

#### Explanation
- **JOIN** `sales` with `menu` to get product details and prices for each order.
- **LEFT JOIN** with `members` to bring in customer membership data (some customers may not be members).
- Use a **CASE** statement to determine membership status at the time of purchase:
	- If `order_date < join_date` → `'N'` (not yet a member).
	- If `order_date >= join_date` → `'Y'` (already a member).
	- Else defaults to `'N'`.
- **ORDER BY customer_id, order_date** arranges the output in chronological order for each customer.

#### Answer: 
| customer_id | order_date | product_name | price | member_status |
| ----------- | ---------- | -------------| ----- | ------------- |
| A           | 2021-01-01 | sushi        | 10    | N      		  |
| A           | 2021-01-01 | curry        | 15    | N      		  |
| A           | 2021-01-07 | curry        | 15    | Y      		  |
| A           | 2021-01-10 | ramen        | 12    | Y      		  |
| A           | 2021-01-11 | ramen        | 12    | Y      		  |
| A           | 2021-01-11 | ramen        | 12    | Y      		  |
| B           | 2021-01-01 | curry        | 15    | N      		  |
| B           | 2021-01-02 | curry        | 15    | N      		  |
| B           | 2021-01-04 | sushi        | 10    | N      		  |
| B           | 2021-01-11 | sushi        | 10    | Y      		  |
| B           | 2021-01-16 | ramen        | 12    | Y      		  |
| B           | 2021-02-01 | ramen        | 12    | Y      		  |
| C           | 2021-01-01 | ramen        | 12    | N     		  |
| C           | 2021-01-01 | ramen        | 12    | N      		  |
| C           | 2021-01-07 | ramen        | 12    | N      		  |

***

**Rank All The Things**

**Danny also requires further information about the `ranking` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null `ranking` values for the records when customers are not yet part of the loyalty program.**

````sql
WITH cte7 AS (
	SELECT
		s.customer_id, s.order_date,
		m.product_name, m.price,
		CASE
			WHEN s.order_date < mb.join_date THEN 'N'
			WHEN s.order_date >= mb.join_date THEN 'Y'
			ELSE 'N'
		END AS member_status
	FROM sales s
	JOIN menu m
		ON s.product_id = m.product_id
	LEFT JOIN members mb
		ON s.customer_id = mb.customer_id
ORDER BY s.customer_id, s.order_date
)
SELECT
	*,
    CASE
		WHEN member_status = 'N' THEN NULL
        ELSE RANK() OVER(PARTITION BY customer_id, member_status ORDER BY order_date)
	END AS ranking
FROM cte7
````

#### Explanation:
- Use a **CTE** (`cte7`) to recreate the sales table with membership status (`Y/N`).
- In the final query, apply a CASE:
	- If ``member_status = 'N'` → set ranking as **NULL** (since Danny doesn’t want ranks for non-member purchases).
	- Otherwise, apply **RANK() OVER(PARTITION BY customer_id, member_status ORDER BY order_date)** to rank each customer’s orders after becoming a member.
- This produces a chronological ranking of purchases for each member, starting from the date they joined.

#### Danny’s Assumption:
Only member purchases should be ranked. All purchases made before joining the program are still shown but have a **NULL ranking**.

#### Answer: 
| customer_id | order_date | product_name | price | member_status | ranking | 
| ----------- | ---------- | -------------| ----- | ------------- |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      		  | NULL
| A           | 2021-01-01 | curry        | 15    | N      		  | NULL
| A           | 2021-01-07 | curry        | 15    | Y      		  | 1
| A           | 2021-01-10 | ramen        | 12    | Y      		  | 2
| A           | 2021-01-11 | ramen        | 12    | Y      		  | 3
| A           | 2021-01-11 | ramen        | 12    | Y      		  | 3
| B           | 2021-01-01 | curry        | 15    | N      		  | NULL
| B           | 2021-01-02 | curry        | 15    | N      		  | NULL
| B           | 2021-01-04 | sushi        | 10    | N      		  | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      		  | 1
| B           | 2021-01-16 | ramen        | 12    | Y      		  | 2
| B           | 2021-02-01 | ramen        | 12    | Y      		  | 3
| C           | 2021-01-01 | ramen        | 12    | N      		  | NULL
| C           | 2021-01-01 | ramen        | 12    | N      		  | NULL
| C           | 2021-01-07 | ramen        | 12    | N      		  | NULL

***








































































