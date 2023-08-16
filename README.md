# DannyMaSQLChallengeWeek1
I started the DannyMa 8 week SQL challenge to practice my SQL skills. The week 1 is about Danny running a diner. This repository contains the solutions for each question asked. I did this using postgresql
## Introduction 
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

sales
menu
members
You can inspect the entity relationship diagram and example data below.
![Screenshot (198)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/04221f3e-522d-4e0e-be59-81bc6d6d1647)
## Case Study Questions 
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do 
 customer A and B have at the end of January?
### Solutions
```
SELECT * FROM dannys_diner.menu;
SELECT * FROM dannys_diner.sales;
SELECT * FROM dannys_diner.members;
```
#### Question 1 the total amount spent by each customer.
```
WITH cte_sales as (SELECT customer_id, sales.product_id, SUM(price) AS Amount_spent, menu.product_name
FROM dannys_diner.sales AS sales
LEFT JOIN dannys_diner.menu AS Menu
ON sales.product_id = menu.product_id
GROUP BY customer_id, sales.product_id, menu.product_name
ORDER BY customer_id)
SELECT customer_id, SUM (amount_spent) AS total_spent
FROM cte_sales
GROUP BY customer_id;

```
![Screenshot (197)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/909acb6b-707e-44aa-b5e2-25082f18b927)
#### Question 1 method 2. I used another method to solve the first question.
```
SELECT customer_id, SUM (amount_spent) AS total_spent
FROM (SELECT customer_id, sales.product_id, SUM(price) AS Amount_spent, menu.product_name
FROM dannys_diner.sales AS sales
LEFT JOIN dannys_diner.menu AS Menu
ON sales.product_id = menu.product_id
GROUP BY customer_id, sales.product_id, menu.product_name
ORDER BY customer_id ) AS New_table
GROUP BY customer_id;
```
![Screenshot (197)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/1bbcf86c-1108-4f6a-b2b1-b7a98bf39abf)
#### Q2 What is the number of days each customer visited? 
```
SELECT customer_id, COUNT(DISTINCT order_date) AS no_of_days
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
![Screenshot (199)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/99ee443c-6c37-40b2-8248-299b805980d4)
#### Q3 what was the first item on the menu purcahsed by the customer?
```
SELECT *
FROM (SELECT customer_id, product_name, product_id, order_date,
RANK () OVER (PARTITION BY customer_id ORDER BY order_date) AS rnk,
ROW_NUMBER () OVER (PARTITION BY customer_id ORDER BY order_date) AS rw
FROM (SELECT customer_id, order_date, sales.product_id, SUM(price) AS Amount_spent, menu.product_name
FROM dannys_diner.sales AS sales
LEFT JOIN dannys_diner.menu AS Menu
ON sales.product_id = menu.product_id
GROUP BY customer_id, sales.product_id, menu.product_name, order_date
ORDER BY customer_id) AS new_table) AS old_table
WHERE rw = 1;
```
![Screenshot (200)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/6fcb8238-9ece-4419-8099-8517f76ad4de)


#### Q4 what is the most purchased item on the menu and how many times was it purchased?
```
SELECT menu.product_name, COUNT (order_date) AS most_purchased
FROM dannys_diner.sales AS sales
LEFT JOIN  dannys_diner.menu AS Menu
ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY COUNT (order_date) DESC
LIMIT 1;
```
![Screenshot (201)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/20cbfe17-3c4c-409e-9bea-ba49033220b8)

#### Q5 which item is the most popular for each customer? */
```
SELECT*
FROM (SELECT customer_id, COUNT (order_date), product_name,
ROW_NUMBER () OVER (PARTITION BY customer_id ORDER BY COUNT (order_date)) AS rnk
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS M
ON s.product_id = m.product_id
GROUP BY product_name, s.customer_id
ORDER BY product_name DESC) AS new_new
WHERE rnk = 1;
```
![Screenshot (202)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/02a3cf25-3990-4911-9310-f3e2236b2530)

#### Q6 which item was purchased first after they became a member?
```
WITH cte AS (SELECT customer_id, order_date, m.product_id, join_date, product_name, price,
Row_number () Over (PARTITION by customer_id ORDER BY order_date) AS rnk
FROM (SELECT s.customer_id, order_date, product_id, join_date
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.members AS mb
ON s.customer_id = mb.customer_id) AS mem_table
LEFT JOIN dannys_diner.menu AS m
ON mem_table.product_id = m.product_id
WHERE order_date >= join_date)

SELECT *
FROM cte
WHERE rnk = 1;
```
![Screenshot (203)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/85066cd0-1af7-4931-b36e-5841031384a7)

#### Q7 which item was purchased just before they became a member
```
WITH cte AS (SELECT customer_id, order_date, m.product_id, join_date, product_name, price,
ROW_NUMBER () Over (PARTITION by customer_id ORDER BY order_date desc) AS rwn,
RANK () OVER (PARTITION by customer_id ORDER BY order_date desc) AS rnk
FROM (SELECT s.customer_id, order_date, product_id, join_date
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.members AS mb
ON s.customer_id = mb.customer_id) AS mem_table
LEFT JOIN dannys_diner.menu AS m
ON mem_table.product_id = m.product_id
WHERE order_date < join_date)

SELECT customer_id, order_date, product_name, rwn, rnk
FROM cte
WHERE rwn = 1;
```
![Screenshot (204)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/b3527054-6833-4783-9313-6b0249cbbe05)

#### Q8 What is the total items and amount spent for each member before they became a member?
```
SELECT s.customer_id, COUNT(product_name) AS total_item, SUM(price) AS total_amount_spent
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.members AS mb
ON s.customer_id = mb.customer_id
inner join dannys_diner.menu AS m ON s.product_id = m.product_id
WHERE join_date > order_date
GROUP BY s.customer_id;
```
![Screenshot (205)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/170fa0bc-7f41-47a4-9ca9-d548196aff69)

#### Q9 If each $1 spent equates to 10 points and sushi hAS a 2x points multiplier, how many points would each customer have?
```
WITH CTE AS (SELECT s.customer_id, product_name, price,
CASE
	WHEN product_name = 'sushi' THEN 2*(price * 10)
	ELSE price * 10
	END AS Purchase_points
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.members AS mb
ON s.customer_id = mb.customer_id
inner join dannys_diner.menu AS m ON s.product_id = m.product_id)

SELECT customer_id, SUM (purchase_points), COUNT(product_name)
FROM cte
GROUP BY customer_id;
```
![Screenshot (208)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/fdb98ebe-31f2-4f83-b083-ae09d6e79e60)

#### Q 10 In the first week after a customer joins the program (including their join date) they earn 2x points 
  ON all items, not just sushi - how many points do customer A and B have at the end of January? */
```
WITH cte AS (SELECT *,
CASE 
WHEN new_date > 1 THEN price * 20
WHEN new_date = 0 THEN price * 20
WHEN new_date > 7 THEN price * 10
WHEN product_name = 'sushi' THEN price * 20
ELSE price * 10
END AS Purchase_points
FROM (SELECT s.customer_id, product_name, order_date, join_date, price, (order_date - join_date) AS new_date
	  FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.members AS mb
ON s.customer_id = mb.customer_id
inner join dannys_diner.menu AS m ON s.product_id = m.product_id
) AS point_table)

SELECT customer_id, SUM(purchase_points), COUNT (product_name)
FROM cte
WHERE order_date BETWEEN '2021-01-01' AND '2021-01-31'
AND customer_id BETWEEN 'A' AND 'B'
GROUP BY customer_id;
   ```
![Screenshot (209)](https://github.com/Mave-t-ech/DannyMaSQLChallengeWeek1/assets/111718556/511dafb6-d586-490c-8e6e-966550b1eb15)

