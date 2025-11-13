# Danny's Diner Sales & Loyalty Program Analysis

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
| Customer		| Total Points (Standard Rules) | Total Points (Jan 2021 w/ 7-Day Bonus) 	| Impact 								|
| ------------- | ----------------------------- | ----------------------------------------- | ------------------------------------- | 
| Customer A 	| 860							| 1020										| 160 extra points due to bonus window.	|
| Customer B 	| 940							| 820										| 120 points lost						|
| Customer C 	| 360							| 0 (Non-member) 							| N/A									| 

### Key Business Insights
- **Effective for Customer A:** The 7-day, 2x point multiplier was highly effective for Customer A, boosting their reward by 59%. This validates using a date-based multiplier to drive strong initial member engagement.
- **Major Flaw for Customer B:** The bonus program actively disincentivized Customer B, causing their total points to **drop by 13%** compared to the standard calculation.
- **Risk of Dissatisfaaction:** The current system punishes members like Customer B whose high spending is split across pre- and post-membership periods. This complexity risks driving away a frequent customer segment.

### Actionable Recommendations
- **Refine Bonus Logic:** For members whose bonus-week purchase is already a 2x item (like Sushi), offer an **alternative, guaranteed benefit** (e.g., a fixed 500-point bonus) to ensure all new members feel rewarded.
- **Simplify Program:** Replace layered multipliers with a simpler **tiered loyalty system** where value and frequency (like Customer B's behavior) automatically unlock consistent, high-value benefits.

---
