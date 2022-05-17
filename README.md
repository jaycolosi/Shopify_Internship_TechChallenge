# Shopify Data Science Intership - Tech Challenge

## Question 1: 

Given some sample data, write a program to answer the following:
On Shopify, we have exactly 100 sneaker shops, and each of these shops sells only one model of shoe. We want to do some analysis of the average order value (AOV). When we look at orders data over a 30 day window, we naively calculate an AOV of $3145.13. Given that we know these shops are selling sneakers, a relatively affordable item, something seems wrong with our analysis.

a. Think about what could be going wrong with our calculation. Think about a better way to evaluate this data.<br>
b. What metric would you report for this dataset?<br>
c. What is its value?<br>

## Q1 Answer: 

When visually investigating the dataset, I noticed a possible few outliers in 'order_amount' so firstly I investigated the amounts. By uploading the "2019 Winter Data Science Intern Challenge Data Set" as a csv file, I utilized Google BigQuery to perform the following SQL queries. 

```
SELECT 
DISTINCT order_amount
FROM `shopify-internship-350418.Shopify_Internship.tech_challenge` 
ORDER BY order_amount DESC
```

There is a range of 90-1760 which then jumps to 25725 to 704000. These outliers will have an impact on the data and when calculating Average. So lets calculate the average while omitting these outliers. 


```
SELECT 
ROUND(AVG(order_amount),2) AS avg_order_amount -- round avg 
FROM `shopify-internship-350418.Shopify_Internship.tech_challenge` 
WHERE 
  order_amount <= 1760 -- will omit outliers 
-- AVG without outliers: 302.58
```

The average with the outliers removed is 302.58. However, using median would be a better metric for this query. By using median it will remove the extreme measurements from the set and give a more realistic amount. 

```
SELECT 
  DISTINCT(PERCENTILE_CONT(order_amount, 0.5) OVER ()) AS median_order_amount 
-- Google BigQuery has no MEDIAN function, but it can be calculated using the PERCENTILE_CONT function.
FROM `shopify-internship-350418.Shopify_Internship.tech_challenge` 
```

After running this new query, the median is 284 and would be a better choice for the data evaluation. 


## Question 2: 

Please use queries to answer the following questions. Paste your queries along with your final numerical answers below.

A. How many orders were shipped by Speedy Express in total?
ANSWER: Speedy Express shipped a total of 54 orders. 

```
SELECT
COUNT(DISTINCT o.orderid) AS num_orders -- counted in case there were any repeat order #s 
FROM 
	Shippers s -- shipper info
JOIN 
	Orders o -- order info
    ON s.shipperid = o.shipperid 
WHERE s.shipperid = 1 -- Speedy Express shipperid = 1
 ```

B. What is the last name of the employee with the most orders?
ANSWER: The last name of the employee with the most orders is "Peacock" with a total of 40 orders. 

```
SELECT
e.lastname, -- pull in last name
COUNT(o.orderID) AS num_orders  -- get total # of orders
FROM 
	Employees e -- employee info
JOIN 
	Orders o -- order info
    ON e.employeeID = o.employeeID 
GROUP BY e.lastname 
ORDER BY num_orders DESC -- order to find the employee with the most orders
```

C. What product was ordered the most by customers in Germany?
ANSWER. The product which was ordered the most by customers in Germany was 'Boston Crab Meat" by total quantity. Additionaly, Gorgonzola Telino was order the more times with a total of 5 orders. 


```
-- Will need to join multiple tables to complete this query 
SELECT
p.productname,
COUNT(DISTINCT o.orderid) AS num_orders, 
SUM(od.quantity) AS total_products
FROM 
	Customers c -- country 
JOIN 
	Orders o -- to get to product id 
    ON c.customerID = o.customerID
JOIN
	OrderDetails od
    ON od.orderid = o.orderid
JOIN
	Products p
    ON p.productid = od.productid
WHERE
	country = 'Germany'
GROUP BY
	p.productname
ORDER BY
	total_products DESC
```
