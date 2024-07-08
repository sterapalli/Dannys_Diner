# Dannys_Diner

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

**2.How many days has each customer visited the restaurant?**

**3.What was the first item from the menu purchased by each customer?**

**4.What is the most purchased item on the menu and how many times was it purchased by all customers?**

**5.Which item was the most popular for each customer?**

**6.Which item was purchased first by the customer after they became a member?**

**7.Which item was purchased just before the customer became a member?**

**8.What is the total items and amount spent for each member before they became a member?**

**9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

**10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

