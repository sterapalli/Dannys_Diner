# Dannys_Diner
##### Credit: https://8weeksqlchallenge.com/case-study-1/

#### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

- sales
- menu
- members

## Question and Solution
 1. What is the total amount each customer spent at the restaurant?


```
SELECT 
  sales.customer_id, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC; 
```
 **Selecting Data:**
- ```sales.customer_id```: Selects the ```customer_id``` from the ```sales``` table.
- ```SUM(menu.price) AS total_sales```: Calculates the sum of the ```price``` from the ```menu``` table for each customer and assigns it the alias ```total_sales```.

**Joining Tables:**
- ```FROM dannys_diner.sales```: Specifies the ```sales``` table from the ```dannys_diner``` database schema.
- ```INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id```: This joins the ```sales``` and ```menu``` tables based on the ```product_id```. This ensures the ```price``` information from the ```menu``` table is associated with the corresponding sales data in the ```sales``` table.

**Grouping Data:**
- ```GROUP BY sales.customer_id```: This groups the results by ```customer_id```. This ensures the ```SUM(menu.price)``` is calculated for each unique  customer. Without ```GROUP BY```, the total sales would be summed for all customers combined.
**Ordering Results:**
- ```ORDER BY sales.customer_id ASC```: This orders the final results by ```customer_id``` in ascending order. This provides a clear organization of the data from the perspective of customer IDs.

**Result**
- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

-------------------------------------------------------------------
**2.How many days has each customer visited the restaurant?**
```
SELECT
customer_id, 
COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id;
```
**Selecting Data:**
- ```customer_id```: Selects the ```customer_id``` from the ```sales``` table.
- ```COUNT(DISTINCT order_date) AS visit_count```:
- ```COUNT```: This function counts the number of rows.
- ```DISTINCT order_date```: This ensures that only unique ```order_date``` values are counted. It removes duplicates,so a customer visiting on the same day is counted only once.
- ```AS visit_count```: This assigns an alias ```visit_count``` to the result of the ```COUNT``` function.
**Grouping Data:**
```GROUP BY customer_id```: This groups the results by ```customer_id```. This ensures the ```COUNT(DISTINCT order_date)```is calculated for each unique customer. Without ```GROUP BY```, the total number of visits would be a single value for all customers combined.

**Result**
| customer_id	| visit_count   |
| ----------- | ------------- |
| A             | 4           |
| B             | 6           |
| C             | 2           |

-------------------------------------------------------------------
#### 3. What was the first item from the menu purchased by each customer?
```
WITH ordered_sales AS (
  SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT 
  customer_id, 
  product_name
FROM ordered_sales
WHERE rank = 1
GROUP BY customer_id, product_name;
```
- ```SELECT```: This clause selects the desired columns for the temporary result set.
  - ```sales.customer_id```: Customer ID from the sales table.
  - ```sales.order_date```: Order date from the sales table.
  -  ```menu.product_name```: Product name from the menu table (joined based on ```sales.product_id```).
  -  ```DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS rank```:
  -  ```DENSE_RANK()```: This window function assigns a rank (position) to each row within a partition.
  -  ```PARTITION BY sales.customer_id```: This partitions the data by ```customer_id```. This ensures ranking is done for each customer independently.
  -  ```ORDER BY sales.order_date```: This orders the rows within each partition by order_date in ascending order.
  -  ```AS rank```: This assigns an alias rank to the result of the - ```DENSE_RANK()``` function.
- ```FROM dannys_diner.sales```: Specifies the sales table from the database schema.
- ```INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id```: Joins the ```sales``` and ```menu``` tables based on - ```product_id``` to retrieve corresponding product names.
** Main Query:**
- ```SELECT```: Selects the desired columns from the ```ordered_sales``` CTE.
  -  ```customer_id```: From the ```ordered_sales``` CTE.
  -  ```product_name```: From the ```ordered_sales``` CTE.
- ```FROM ordered_sales```: References the previously defined CTE (```ordered_sales```).
- ```WHERE rank = 1```: Filters the results within the CTE to include only rows where ```rank``` is 1. This selects the row with the earliest ```order_date ```(first purchase) for each customer based on the ranking.
- ```GROUP BY customer_id, product_name:``` This might seem unnecessary here because the ```WHERE``` clause already filters by ```rank = 1```. However, it ensures proper output formatting, especially if there might be duplicate ```product_name``` entries for the same customer on the first purchase date (e.g., two of the same item)

**Result**
| customer_id | product_name |
| ----------- | ------------ |
| A	          | curry        |
| A 	         | sushi        |
| B           | curry        |
| C	          | ramen        |

-------------------------------------------------------------------
**4.What is the most purchased item on the menu and how many times was it purchased by all customers?**
```
SELECT 
  menu.product_name,
  COUNT(sales.product_id) AS most_purchased_item
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY most_purchased_item DESC
LIMIT 1;
```
 ** Selecting Data:**
- ```menu.product_name```: Selects the product name from the menu table.
- ```COUNT(sales.product_id) AS most_purchased_item```:
  -  ```COUNT```: This function counts the number of rows.
  -  s```ales.product_id```: This specifies the column to be counted (number of times each product ID appears in sales).
  -  ```AS most_purchased_item```: This assigns an alias ```most_purchased_item``` to the result of the ```COUNT``` function.
**Joining Tables:**
- ```FROM dannys_diner.sales```: Specifies the sales table from the database schema.
- ```INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id```: Joins the ```sales``` and ```menu``` tables based on ```product_id``` to associate product names with corresponding sales data.
**Grouping Data:**
- ```GROUP BY menu.product_name```: This groups the results by ```product_name```. This ensures the ```COUNT(sales.product_id)``` is calculated for each unique product. Without ```GROUP BY```, the total count would be for all products combined.
**Ordering and Limiting Results:**
- ```ORDER BY most_purchased_item DESC```: Orders the results by ```most_purchased_item``` in descending order (most popular first).
- ```LIMIT 1```: Limits the retrieved results to only the first row (the product with the highest count).

**Result**
|product_name	|most_purchased_item |
| ----------- | ------------------ |
| ramen	      |        8           |

-------------------------------------------------------------------
**5.Which item was the most popular for each customer?**
```
WITH most_populor_item AS (
	SELECT sales.customer_id, menu.product_name,
	COUNT(menu.product_id) AS order_count,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id 
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  	ON sales.product_id = menu.product_id
  GROUP BY sales.customer_id, menu.product_name
)
SELECT 
  customer_id, 
  product_name, 
  order_count
FROM most_populor_item 
WHERE rank = 1;
```
This code snippet utilizes a Common Table Expression (CTE) to identify the most popular item for each customer at Danny's Diner. Here's a breakdown:
 **Common Table Expression (CTE):**
- ```WITH most_popular_item AS ( ... )```: This defines a CTE named ```most_popular_item```. It acts as a temporary named result set used within the main query.
Within the CTE:
- ```SELECT```: This clause selects the desired columns for the temporary result set.
  -  ```customer_id, product_name, COUNT(*) AS order_count```: These columns are essential to identify the most popular item for each customer.
  -  ```COUNT(*) AS order_count```: Counts all purchases for each product by customer.
  -  ```DENSE_RANK()``` assigns ranks based on these counts within each customer group.
- ```FROM dannys_diner.sales```: Specifies the ```sales``` table from the database schema.
- ```JOIN dannys_diner.menu ON sales.product_id = menu.product_id```: Joins the ```sales``` and ```menu``` tables based on ```product_id``` to retrieve corresponding product names.
- ```GROUP BY sales.customer_id, menu.product_name```: Groups the results by both ```customer_id``` and ```product_name```. This ensures ```order_count``` is calculated for each unique combination of customer and product.

 **Main Query:**
- ```SELECT```: Selects the desired columns from the ```most_popular_item``` CTE.
 -  ```customer_id```: From the ```most_popular_item```CTE.
  -  ```product_name: From the ```most_popular_item``` CTE.
  -  ```order_count```: From the ```most_popular_item``` CTE.
- ```FROM most_popular_item```: References the previously defined CTE (most_popular_item).
- ```WHERE rank = 1```: Filters the results within the CTE to include only rows where rank is 1. This selects the product with the highest ```order_count``` (most frequent purchase) for each customer.

**Result**
|customer_id	| product_name	| order_count|
|------------|--------------|------------|
| A	         |  ramen	      | 3          |
| B	         |  ramen	      | 2          |
| B	         |  curry	      | 2          |
| B	         |  sushi	      | 2          |
| C	         |  ramen	      | 3          |

-------------------------------------------------------------------
**6.Which item was purchased first by the customer after they became a member?**

```
WITH joined_as_member AS (
  SELECT
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER(
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS row_num
  FROM dannys_diner.members
  JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date > members.join_date
)

SELECT 
  customer_id, 
  product_name 
FROM joined_as_member
JOIN dannys_diner.menu
  ON joined_as_member.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```
This code effectively finds the first item purchased by each customer after joining Danny's Diner. Here's a breakdown of its functionality:

**Common Table Expression (CTE):**

- ```joined_as_member``` CTE identifies sales for members after their join date.
- It joins the ```members``` and ```sales``` tables based on ```customer_id```.
- Filters for sales where ```order_date``` is greater than ```join_date```.
- Assigns a sequential number (```row_num```) to each sale record within each customer group (```ordered by order_date```).
**Main Query:**

- ```Selects customer_id``` and ```product_name``` from the ```oined_as_member CTE``` and ```menu``` table, respectively.
  -  Joins the CTE with the ```menu``` table using ```product_id``` to retrieve product names.
- Filters for rows where ```row_num = 1``` (selecting the first purchase for each customer).
- Orders the results by ```customer_id``` in ascending order.
**Result**
| customer_id	| product_name |
| A           | ramen        |
| B           | sushi        |

---------------------------------
**7.Which item was purchased just before the customer became a member?**
```
-- 7. Which item was purchased just before the customer became a member?


WITH joined_as_member AS (
  SELECT
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER(
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS row_num
  FROM dannys_diner.members
  JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date < members.join_date
)

SELECT 
  customer_id, 
  product_name 
FROM joined_as_member
JOIN dannys_diner.menu
  ON joined_as_member.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```

**Common Table Expression (CTE):**

- ```joined_as_member``` CTE identifies sales for members before their join date.
  -  It joins the ```members``` and ```sales``` tables based on ```customer_id```.
 -  Filters for sales where order_date is less than join_date (correct for identifying purchases before joining).
 -  Assigns a sequential number (```row_num```) to each sale record within each customer group (ordered by ```order_date```). Here's the slight change:
- We need the highest order number (latest purchase) before joining, not the first (lowest). We can achieve this by changing ```ORDER BY sales.order_date to ORDER BY sales.order_date DESC```.
   **Main Query (remains unchanged):**

- Selects ```customer_id``` and ```product_name``` from the ```joined_as_member``` CTE and ```menu``` table, respectively.
- Joins the CTE with the ```menu``` table using ```product_id``` to retrieve product names.
- Filters for rows where ```row_num = 1``` (selecting the latest purchase for each customer before joining).
- Orders the results by ```customer_id``` in ascending order.

**Result**
| customer_id	| product_name |
|-------------|--------------|
| A           | sushi        |
| B           | curry        |

----------------------------------------
**8.What is the total items and amount spent for each member before they became a member?**

```
SELECT 
  sales.customer_id, 
  COUNT(sales.product_id) AS total_items, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  AND sales.order_date < members.join_date
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

**Joining Tables:**

- ```FROM dannys_diner.sales```: Specifies the ```sales``` table from the database schema.
- ```JOIN dannys_diner.members ON sales.customer_id = members.customer_id```: Joins the ```sales``` and ```members``` tables based on ```customer_id```. This ensures we only consider sales data for customers who eventually became members.
- ```AND sales.order_date < members.join_date```: Filters the joined data to include only sales that occurred before the customer's join date. This focuses on purchases made while they were not yet members.
- ```JOIN dannys_diner.menu ON sales.product_id = menu.product_id```: Joins the ```sales``` table with the ```menu``` table based on ```product_id``` to retrieve corresponding product prices for calculating total sales.
 **Grouping and Aggregating Data:**

- ```GROUP BY sales.customer_id```: Groups the results by ```customer_id```. This ensures separate calculations of total items and total sales for each member.
- ```COUNT(sales.product_id) AS total_items```: Counts the number of product IDs (number of items purchased) for each customer group.
- ```SUM(menu.price) AS total_sales```: Calculates the sum of ```menu.price``` (total amount spent) for each customer group based on their purchases before joining.
 **Ordering Results:**

- ```ORDER BY sales.customer_id```: Orders the final results by ```customer_id``` in ascending order, providing information for each member in a predictable order.

**Result**

|customer_id	| total_items	| total_sales |
|------------|-------------|-------------|
| A          |	2	          | 25          |
| B          |	3	          | 40          |

**9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```
WITH points_cte AS (
  SELECT 
    menu.product_id, 
    CASE
      WHEN product_id = 1 THEN price * 20
      ELSE price * 10
    END AS points
  FROM dannys_diner.menu
)

SELECT 
  sales.customer_id, 
  SUM(points_cte.points) AS total_points
FROM dannys_diner.sales
JOIN points_cte
  ON sales.product_id = points_cte.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

Common Table Expression (CTE):
- ```points_cte``` CTE defines a named result set to calculate points based on product ID.
  -  It selects ```product_id``` and ```price``` from the ```menu``` table.
  -  The ```CASE``` statement assigns points based on a rule:
   -   Products with ```product_id = 1``` earn double points (```price * 20```).
   -   All other products earn regular points (```price * 10```).
Main Query:
- Selects ```customer_id``` from the ```sales``` table and ```SUM(points_cte.points)``` as total_points.
- Joins the ```sales``` table with the ```points_cte``` CTE on ```product_id``` to associate points with each sale.
- ```GROUP BY sales.customer_id```: Groups the results by customer, ensuring the total points are calculated for each individual.
- ```ORDER BY sales.customer_id```: Orders the final results by ```customer_id``` in ascending order.

**Results**
|customer_id	|total_points |
|------------|-------------|
| A          |	860         |
| B          |	940         |
| C          |	360         |

----------------------------------

**10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
 ```
WITH dates_cte AS (
  SELECT 
    customer_id, 
    join_date, 
    join_date + 6 AS valid_date, 
    DATE_TRUNC(
      'month', '2021-01-31'::DATE)
      + interval '1 month' 
      - interval '1 day' AS last_date
  FROM dannys_diner.members
)

SELECT 
  sales.customer_id, 
  SUM(CASE
    WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price
    WHEN sales.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 * menu.price
    ELSE 10 * menu.price END) AS points
FROM dannys_diner.sales
JOIN dates_cte AS dates
  ON sales.customer_id = dates.customer_id
  AND sales.order_date <= dates.last_date
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id;
```

**Common Table Expression (CTE):**

- ```dates_cte``` CTE defines calculations related to join dates and valid periods.
- Selects relevant columns from the ```members``` table.
- ```join_date + 6 AS valid_date```: Calculates the end date of the first week after joining (assuming a 7-day week).
- ```DATE_TRUNC('month', '2021-01-31'::DATE) + interval '1 month' - interval '1 day' AS last_date```: This calculates the last day of January 2021. We use DATE_TRUNC to get the first day of the month, then add a month and subtract a day to get the last day.
**Main Query:**

- Selects ```customer_id``` and calculates total points with a ```SUM``` and a ```CASE``` statement.
- Joins the ```sales``` table with the ```dates_cte``` CTE on ```customer_id``` to associate join dates and valid periods with each sale.
- Filters the joined data using ```sales.order_date <= dates.last_date``` to ensure only sales within January 2021 are considered.
- Joins the ```sales``` table with the ```menu``` table on ```product_id``` to retrieve product prices.
- The ```CASE``` statement assigns points based on conditions:
- Double points for sushi purchases (```menu.product_name = 'sushi'```).
- Double points during the first week after joining (```sales.order_date BETWEEN dates.join_date AND dates.valid_date```).
- Regular points (10 points per dollar) for all other cases.
- ```GROUP BY sales.customer_id```: Groups the results by customer, ensuring separate point calculations for each.

  **Result**
|customer_id |	points|
|------------|-------|
| A          |	1370  |
| B          |	820   |

-----------------------------------
