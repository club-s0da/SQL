```sql
--#Querying a database with multiple tables to quantify statistics about customer and order data. 

--#Data cleaning: filters out rows where order id is null, blank, not 6 characters, and a specific string.
SELECT *
  FROM BIT_DB.JanSales
 WHERE length(order_id) = 6 AND 
       order_id <> 'Order ID' AND 
       order_id IS NOT NULL AND 
       order_id <> '';
       
-- #Data exploration; visualizing what columns are in the data.
SELECT *
  FROM BIT_DB.JanSales
 LIMIT 20;
 
 -- #Q1: How many orders were placed in January?
SELECT COUNT(orderid) 
  FROM BIT_DB.JanSales
 WHERE length(orderid) = 6 AND 
       orderid <> 'Order ID';
       
-- #Q2: How many of those orders were for an iPhone?
SELECT COUNT(orderid) 
  FROM BIT_DB.JanSales
 WHERE length(orderid) = 6 AND 
       orderid <> 'Order ID' AND 
       product = 'iPhone';
       
-- #Q3: Select the customer account numbers for all the orders that were placed in February.
SELECT DISTINCT cust.acctnum
  FROM BIT_DB.customers cust
       INNER JOIN
       BIT_DB.FebSales feb ON cust.order_id = feb.orderid
 WHERE length(order_id) = 6 AND 
       order_id <> 'Order ID';
       
-- #Q4: Which product was the cheapest one sold in Janurary, and what was the price?
SELECT DISTINCT Product,
                price
  FROM BIT_DB.JanSales
 WHERE price IN (
           SELECT min(price) 
             FROM BIT_DB.JanSales
       );
-- alternatively:
SELECT DISTINCT Product,
                price
  FROM BIT_DB.JanSales
 ORDER BY price ASC
 LIMIT 1;
 -- alternitively:
SELECT DISTINCT Product,
                min(price) 
  FROM BIT_DB.JanSales
 GROUP BY Product,
          price
 ORDER BY price ASC
 LIMIT 1;
 -- alternatively:
SELECT Product,
       min(price) 
  FROM BIT_DB.JanSales
 GROUP BY Product,
          price
 ORDER BY price ASC
 LIMIT 1;
 
-- #Q5: What is the total revenue for each product sold in January?
SELECT Product,
       (sum(Quantity) * price) AS janRevenue
  FROM BIT_DB.JanSales
 GROUP BY Product;
 
-- #Q6: Which products were sold in February at 548 Lincoln St, Seattle, WA 98101, how many of each were sold, and what was the total revenue?
SELECT sum(quantity),
       product,
       sum(quantity) * price AS revenue
  FROM BIT_DB.FebSales
 WHERE location = '548 Lincoln St, Seattle, WA 98101'
 GROUP BY product;
 
-- #Q7: How many customers ordered more than 2 products at a time in February, and what was the average amount spent for those customers?
SELECT COUNT(DISTINCT cust.acctnum),
       feb.quantity,
       avg(feb.quantity * feb.price) AS avgSpent
  FROM BIT_DB.FebSales feb
       LEFT JOIN
       BIT_DB.customers cust ON cust.order_id = feb.orderid
 WHERE feb.quantity > 2 AND 
       length(order_id) = 6 AND 
       order_id <> 'Order ID';

--#Q8: List all the products sold in Los Angeles in February, and include how many of each were sold.
SELECT Product,
       sum(Quantity) 
  FROM BIT_DB.FebSales
 WHERE location LIKE '%Los Angeles%'
 GROUP BY Product;

-- #Q9: Which locations in New York received at least 3 orders in January, and how many orders did they each receive?
SELECT DISTINCT location,
                orderID,
                count(orderID) AS [Number of Orders]
  FROM BIT_DB.JanSales
 WHERE location LIKE '%NY%'
 GROUP BY location
HAVING count(orderID) >= 3;

 -- #Q10: How many of each type of headphone were sold in February?
SELECT sum(quantity) AS Quantity,
       Product
  FROM BIT_DB.FebSales
 WHERE Product LIKE '%headphone%'
 GROUP BY Product;

-- #Q11: What was the average amount spent per account in February?
-- Note to self: no GROUP BY statement even tho contains aggregation bc we are looking for one value anyway (singular aggregation).
SELECT acctnum,
       sum(quantity * price) / count(acctnum) AS [Avg Spending per Account]
  FROM BIT_DB.FebSales feb
       LEFT JOIN
       BIT_DB.customers cust ON feb.orderID = cust.order_ID
 WHERE length(order_id) = 6 AND 
       order_id <> 'Order ID';
-- alternatively:
SELECT avg(quantity * price) 
  FROM BIT_DB.FebSales feb
       LEFT JOIN
       BIT_DB.customers cust ON feb.orderID = cust.order_ID
 WHERE length(order_id) = 6 AND 
       order_id <> 'Order ID';
       
-- #Q12: What was the average quantity of products purchased per account in February?
SELECT sum(quantity) / count(acctnum) AS avgQuantityPerAcct
  FROM BIT_DB.FebSales feb
       LEFT JOIN
       BIT_DB.customers cust ON feb.orderID = cust.order_ID
 WHERE length(order_id) = 6 AND 
       order_id <> 'Order ID';

 -- #Q13: Which product brought in the most revenue in Janurary and how much revenue did it bring in total?
SELECT Product,
       sum(quantity * price) AS revenue
  FROM BIT_DB.JanSales
 GROUP BY Product
 ORDER BY revenue DESC
 LIMIT 1;
```
