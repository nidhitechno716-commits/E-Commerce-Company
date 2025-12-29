# E-Commerce-Company

## Project Overview
**Project Title**: E-Commerce Analysis    

**Database**: `ecommerce`

e-commerce company databases to extract insights that drive business strategies forward.analysis will inform various departments, from marketing to supply chain, providing them with actionable data to optimize our operations, enhance customer satisfaction, and boost our sales performance
## Objectives
1. **Set up a retail sales database**: Create and populate a ecommerce database with the provided customres,products,orders and order details data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the  data.
5. 
## Project Structure

### 1. Database Setup
- **Database Creation**: The project starts by creating a database named `ecommerce`.
- **Table Creation**: There are total 4 table we need to create
- A table named `customres` is created to store the  customeres data. The table structure includes columns for customer_id	name	location
- A table named `products`  is created to store the roduct data.The table structure includes colums for product_id	name	category	price
- A table named `orders`  is created to store the roduct data.The table structure includes colums for order_id	order_date	customer_id	total_amount
- A table named `orderdetails`  is created to store the roduct data.The table structure includes colums for order_id	product_id	quantity	price_per_unit
```sql
CREATE DATABASE ecommerce;
CREATE TABLE customers (
    customer_id INT NOT NULL,
    customer_name VARCHAR(45),
    location VARCHAR(45),
    PRIMARY KEY (customer_id)
);
CREATE TABLE products (
    product_id INT NOT NULL,
    product_name VARCHAR(45),
    category VARCHAR(45),
    price INT,
    PRIMARY KEY (product_id)
);
CREATE TABLE orders (
    order_id INT NOT NULL,
    order_date DATE,
    customer_id INT,
    total_amount INT,
    PRIMARY KEY (order_id),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
CREATE TABLE order_details (
    order_detail_id INT NOT NULL AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT,
    price_per_unit INT,
    PRIMARY KEY (order_detail_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```
### 2. Data Exploration & Cleaning

- **products Count**: Determine the total number of products in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify  unique categories in the dataset.
  
### 3. Data Analysis & Findings

1 **Identify the top 3 cities with the highest number of customers to determine key markets for targeted marketing and logistic optimization**.
```sql
SELECT 
    location,
    COUNT(customer_id) AS number_of_customers
FROM 
    Customers
GROUP BY 
    location
ORDER BY 
    number_of_customers DESC
LIMIT 3;
```
2 **Determine the distribution of customers by the number of orders placed. This insight will help in segmenting customers into one-time buyers, occasional shoppers, and regular customers for tailored marketing strategies**
```sql
SELECT 
    CASE
        WHEN order_count = 1 THEN 1
        WHEN order_count BETWEEN 2 AND 4 THEN order_count  -- or a range like '2-4'
        WHEN order_count > 4 THEN order_count
    END AS numberOfOrders,
    COUNT(*) AS CustomerCount
FROM (
    SELECT 
        customer_id,
        COUNT(order_id) AS order_count
    FROM 
        Orders
    GROUP BY 
        customer_id
) AS customer_summary
GROUP BY 
    numberOfOrders
ORDER BY 
    numberOfOrders;
```
3.**Identify products where the average purchase quantity per order is 2 but with a high total revenue, suggesting premium product trends.**
```sql
SELECT
    product_id,
    AVG(quantity) AS AvgQuantity,
    SUM(quantity * price_per_unit) AS TotalRevenue
FROM
    OrderDetails
GROUP BY
    product_id
HAVING
    AVG(quantity) = 2  
ORDER BY
    TotalRevenue DESC
```
4.**For each product category, calculate the unique number of customers purchasing from it. This will help understand which categories have wider appeal across the customer base.**
```sql
SELECT
    p.category,
    COUNT(DISTINCT o.customer_id) AS unique_customers
FROM
    Orders o
JOIN
    OrderDetails od ON o.order_id = od.order_id
JOIN
    Products p ON od.product_id = p.product_id
GROUP BY
    p.category
ORDER BY
    unique_customers DESC;
```
5.**Analyze the month-on-month percentage change in total sales to identify growth trends**
```sql
SELECT
    DATE_FORMAT(order_date, '%Y-%m') AS month,
    SUM(total_amount) AS TotalSales,
    ROUND(
        (SUM(total_amount) - LAG(SUM(total_amount)) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m'))) 
        / LAG(SUM(total_amount)) OVER (ORDER BY DATE_FORMAT(order_date, '%Y-%m')) * 100, 
    2) AS PercentChange
FROM
    Orders
GROUP BY
    DATE_FORMAT(order_date, '%Y-%m')
ORDER BY
    month;
```
6.**Examine how the average order value changes month-on-month. Insights can guide pricing and promotional strategies to enhance order value.**
```sql
WITH MonthlyAvg AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS Month,
        ROUND(AVG(total_amount), 2) AS AvgOrderValue
    FROM Orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT 
    Month,
    AvgOrderValue,
    ROUND(AvgOrderValue - LAG(AvgOrderValue) OVER (ORDER BY Month), 2) AS ChangeInValue
FROM MonthlyAvg
ORDER BY ChangeInValue DESC;
```
7.**Based on sales data, identify products with the fastest turnover rates, suggesting high demand and the need for frequent restocking.**
```sql
SELECT 
    product_id,
    COUNT(order_id) AS SalesFrequency
FROM OrderDetails
GROUP BY product_id
ORDER BY SalesFrequency DESC
LIMIT 5;
```
8.**List products purchased by less than 40% of the customer base, indicating potential mismatches between inventory and customer interest**
```sql
WITH ProductCustomerCount AS (
    SELECT 
        od.product_id,
        COUNT(DISTINCT o.customer_id) AS UniqueCustomerCount
    FROM OrderDetails od
    JOIN Orders o ON od.order_id = o.order_id
    GROUP BY od.product_id
),
TotalCustomerBase AS (
    SELECT COUNT(DISTINCT customer_id) AS TotalCustomers
    FROM Customers
)
SELECT 
    p.product_id as Product_id,
    p.name AS Name,
    pc.UniqueCustomerCount
FROM ProductCustomerCount pc
JOIN Products p ON pc.product_id = p.product_id
CROSS JOIN TotalCustomerBase t
WHERE pc.UniqueCustomerCount < 0.4 * t.TotalCustomers
ORDER BY pc.UniqueCustomerCount ASC;
```
9.**Evaluate the month-on-month growth rate in the customer base to understand the effectiveness of marketing campaigns and market expansion efforts.**
```sql
SELECT
    DATE_FORMAT(first_purchase_date, '%Y-%m') AS FirstPurchaseMonth,
    COUNT(*) AS TotalNewCustomers
FROM (
    SELECT
        customer_id,
        MIN(order_date) AS first_purchase_date
    FROM Orders
    GROUP BY customer_id
) AS first_orders
GROUP BY DATE_FORMAT(first_purchase_date, '%Y-%m')
ORDER BY FirstPurchaseMonth ASC;
```
10.**Identify the months with the highest sales volume, aiding in planning for stock levels, marketing efforts, and staffing in anticipation of peak demand period.**
```sql
SELECT
    DATE_FORMAT(order_date, '%Y-%m') AS Month,
    SUM(total_amount) AS TotalSales
FROM Orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY TotalSales DESC
LIMIT 3;
```
## Findings

### Engagement Depth Analysis : 

1.As the Number of orders increases, the Customer count decreases. 
2. Occasional Shoppers experiences the most 

### Category-wise Customer Reach
1.Electronics category needs more focus as it is in high demand among the customers

### Sales Trend Analysis 
1.As per Sales Trend Analysis the sales  month Feb 2024 experience the largest decline.
2.sales trend from March to August Sales fluctuated with no clear trend.

### month-on-month Trends
1.December month has the highest change in the average order value

### Peak Sales Period Identification
1.September and December months will require major restocking of product and increased staffs

### Customer Acquisition Trends
1.It is downward trend which implies the marketing campaign are not much effective.
2.strategic action to improve the sales of these underperforming products is  Implement targeted marketing campaigns to raise awareness and interest.

## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.
## Author - Nidhi Desai 

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. 

