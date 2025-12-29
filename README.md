# E-Commerce-Company

## Project Overview
**Project Title**: E-Commerce Analysis    
**Database**: `ecommerce`
e-commerce company databases to extract insights that drive business strategies forward.analysis will inform various departments, from marketing to supply chain, providing them with actionable data to optimize our operations, enhance customer satisfaction, and boost our sales performance
## Objectives
1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.
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
1 **Identify the top 3 cities with the highest number of customers to determine key markets for targeted marketing and logistic optimization **.
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
