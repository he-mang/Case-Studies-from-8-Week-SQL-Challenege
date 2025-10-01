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
    SUM(m.price) AS total_price
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY customer_id;
````
