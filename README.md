create database amazon_fresh; 
use amazon_fresh;
select* from amazon_fresh.customers;
select * from amazon_fresh.customers   
-- Task 3
WHERE City = 'Port Calebstad';
SELECT * FROM amazon_fresh.products
WHERE Category = 'Fruits';

-- Task 4
CREATE TABLE customers (customerid INT PRIMARY KEY,name VARCHAR(100) UNIQUE,age INT NOT NULL CHECK(age > 18),city VARCHAR(100));
SELECT * FROM amazon_fresh.products;
INSERT INTO amazon_fresh.products(ProductID,ProductName,Category,SubCategory,PricePerUnit,StockQuantity,SupplierID)VALUES('P102','Management','Event','Sub-Event',1000,100,'03ec3130-f542-432e-b173-f11032469026');
SELECT * FROM amazon_fresh.suppliers;

-- Task 5
INSERT INTO amazon_fresh.products
(ProductID, ProductName, Category, SubCategory, PricePerUnit, StockQuantity, SupplierID)VALUES
('3223799f-123v-23a5-99d3-234as424d84g','watermelon','Event','Sub-Event',1000,100,'03ec3130-f542-432e-b173-f11032469026');
INSERT INTO amazon_fresh.products
(ProductID, ProductName, Category, SubCategory, PricePerUnit, StockQuantity, SupplierID)VALUES
('3425fe4f-113v-23e9-10d3-23sus12d45f5','Apple','Fruits','Fresh Fruits',1000,1000,'03553210-n5a2-2121-a313-n11032469023'); 
INSERT INTO amazon_fresh.products
(ProductID, ProductName, Category, SubCategory, PricePerUnit, StockQuantity, SupplierID)VALUES
('43h19gf5-123d-27a5-hhd3-234ab1b004g48','Sports','event','Sub-Event',500,200,'03111130-711f-111w-3431-h54324690261');

-- Task 6
UPDATE amazon_fresh.products
SET StockQuantity = 500
WHERE ProductID = '0006B53D-74cb-44a2-91cd-69d0a31c5b5b'; 

-- Task 7

DELETE FROM amazon_fresh.suppliers
WHERE SupplierID = '03ec3130-f542-432e-b173-f11032469026';

SELECT * FROM amazon_fresh.suppliers;
DESCRIBE amazon_fresh.products;     

-- Task 8

select * from amazon_fresh.reviews;
alter table amazon_fresh.reviews add constraint chk_rating check (rating between 1 and 5);
ALTER TABLE amazon_fresh.customers ADD COLUMN df_primemeber VARCHAR(10) DEFAULT 'NO';


-- Task 9

-- WHERE clause to find orders placed after 2021-01-01

SELECT * FROM amazon_fresh.customers;
SELECT * FROM amazon_fresh.orders;
SELECT *FROM amazon_fresh.orders
WHERE orderDate > '2021-01-01';

-- HAVING clause to list products with average rating greater than 4

SELECT * FROM amazon_fresh.products;
SELECT * FROM amazon_fresh.reviews;
SELECT ProductID,AVG(rating) AS averagerating FROM amazon_fresh.reviews
GROUP BY ProductID HAVING AVG(rating) > 4;

-- GROUP BY and ORDER BY clauses to rank products by total sales

SELECT ProductID,SUM(StockQuantity * PricePerUnit) AS totalsales FROM amazon_fresh.products
GROUP BY ProductID ORDER BY totalsales DESC;

-- Task 10 – Identifying High-Value Customers

ALTER TABLE customers ADD total_spend DECIMAL(10,2); 

SHOW TABLES; 

-- 1 Calculate each customer's total spending ..!!!

USE amazon_fresh;

SELECT o.CustomerID,SUM(od.Quantity * p.PricePerUnit) AS TotalSpending FROM amazon_fresh.orders o
JOIN order_details od ON o.OrderID = od.OrderID
JOIN products p ON od.ProductID = p.ProductID
GROUP BY o.CustomerID;

-- 2  Rank customers based on their spending 

SELECT o.CustomerID,SUM(od.Quantity * p.PricePerUnit) AS TotalSpending FROM amazon_fresh.orders o
JOIN amazon_fresh.order_details od ON o.OrderID = od.OrderID
JOIN amazon_fresh.products p ON od.ProductID = p.ProductID
GROUP BY o.CustomerID
ORDER BY TotalSpending DESC;

-- 3 Identify customers who have spent more than 5000

SELECT o.CustomerID,SUM(od.Quantity * p.PricePerUnit) AS TotalSpending FROM amazon_fresh.orders o
JOIN amazon_fresh.order_details od ON o.OrderID = od.OrderID
JOIN amazon_fresh.products p ON od.ProductID = p.ProductID
GROUP BY o.CustomerID
HAVING SUM(od.Quantity * p.PricePerUnit) > 5000;


-- Task 11 – Aggregations and Joins

-- Join the orders and order details tables to calculate total revenue per order 
SELECT o.OrderID,SUM(od.Quantity * p.PricePerUnit) AS TotalRevenue FROM amazon_fresh.orders AS o
JOIN amazon_fresh.order_details AS od ON o.OrderID = od.OrderID
JOIN amazon_fresh.products AS p ON od.ProductID = p.ProductID
GROUP BY o.OrderID;  

-- Identify customers who placed the most orders in a specific time period
SELECT CustomerID,COUNT(OrderID) AS TotalOrders 
FROM amazon_fresh.orders
WHERE OrderDate BETWEEN '2025-01-01' AND '2025-12-31'
GROUP BY CustomerID
ORDER BY TotalOrders DESC;

-- Find the supplier with the highest stock 
SELECT SupplierID,SUM(StockQuantity) AS TotalStock
FROM amazon_fresh.products
GROUP BY SupplierID
ORDER BY TotalStock DESC
LIMIT 1;




-- Task 12 
-- Separate Categories & Subcategories into a New Table

USE amazon_fresh;

CREATE TABLE Categories (CategoryID VARCHAR(100) PRIMARY KEY,Category VARCHAR(100),SubCategory VARCHAR(100));
INSERT INTO Categories (CategoryID, Category, SubCategory)SELECT DISTINCT UUID(), Category, SubCategory FROM amazon_fresh.products;

-- Create foreign keys to maintain relationships

ALTER TABLE amazon.products
ADD CategoryID VARCHAR(100);

SET SQL_SAFE_UPDATES = 0;

UPDATE amazon.products p JOIN Categories c ON p.Category = c.Category AND p.SubCategory = c.SubCategory SET p.CategoryID = c.CategoryID;

-- Create foreign keys to maintain relationships

SHOW TABLES;

DESC Categories;

ALTER TABLE amazon.products ADD CONSTRAINT fk_category FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID);

-- Create Categories table

CREATE TABLE amazon.Categories (CategoryID VARCHAR(100) PRIMARY KEY,Category VARCHAR(100),SubCategory VARCHAR(100));


INSERT INTO amazon.Categories (CategoryID, Category, SubCategory)
SELECT DISTINCT UUID(),Category,SubCategory FROM products;

ALTER TABLE amazon.products ADD CONSTRAINT ff_category FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID);

SET SQL_SAFE_UPDATES = 0;

UPDATE amazon.products p JOIN amazon.Categories c ON p.Category = c.Category AND p.SubCategory = c.SubCategory
SET p.CategoryID = c.CategoryID;

-- Task 13

SELECT Productid ,Total_Revenue 
from (select productID,sum((UnitPrice * Quantity)- discount) as total_revenue from amazon_fresh.order_details
    group by productID) as revenue order by total_revenue desc limit 3;
 select CustomerID,name from customers where CustomerID not in (select CustomerID from orders);
 
 
-- Task 14 : Provide Actionable Insights

-- Which cities have the highest concentration of Prime Members?

SELECT City, COUNT(*) AS PrimeMembers FROM amazon_fresh.customers 
WHERE PrimeMember = 'Yes' GROUP BY City ORDER BY PrimeMembers DESC;


-- What are the Top 3 Most Frequently Ordered Categories?

SELECT p.Category,COUNT(oi.ProductID) AS Total_Orders FROM amazon_fresh.order_details AS oi INNER JOIN amazon_fresh.products AS p
ON oi.ProductID = p.ProductID GROUP BY p.Category ORDER BY Total_Orders DESC LIMIT 3;


-- View Customers Table
SELECT * FROM amazon_fresh.customers;
SELECT * FROM amazon_fresh.order_details;
SELECT * FROM amazon_fresh.orders;
SELECT * FROM amazon_fresh.products;
SELECT * FROM amazon_fresh.reviews;
SELECT * FROM amazon_fresh.suppliers;
