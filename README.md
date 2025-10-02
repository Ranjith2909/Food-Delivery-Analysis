# Food Delivery Analysis

A project analyzing food delivery data to understand trends, customer preferences, and delivery performance using SQL, Python, and Power BI.

## Features
- Data cleaning and preparation
- Exploratory Data Analysis (EDA)
- Visualizations and dashboards
- Insights for improving delivery efficiency

## Technologies
- SQL
- Power BI

## Author
Ranjith Kumar

#1. Project Description

#---This project simulates a food delivery platform (like Zomato or Swiggy) using SQL. 
#----The goal is to design a relational database that tracks customers, 
#---restaurants, menu items, and orders. With this data you can run analytics such as order frequency,
#--- revenue per restaurant, popular items, and payment trends.

#2. About

#Food delivery apps connect customers with restaurants, let them browse menus, place orders, and pay online.
 #A backend database stores all the interactions — customers, orders, delivery times, and ratings.

#Table Name	-- -Purpose
#customers-----	Stores customer details.
#restaurants----Stores restaurant information.
#menu_items	----Stores items offered by each restaurant (FoodItem, price).
#orders	--------Tracks each order’s metadata (order date, delivery time, payment, rating).
#order_items---	Tracks which items and quantities belong to which order.

create database food;
use food;
select * from food_data;

UPDATE food_data
SET CustomerName = 'Unknown'
WHERE CustomerName IS NULL;

UPDATE food_data
SET RestaurantName = 'Unknown'
WHERE RestaurantName IS NULL;

UPDATE food_data
SET FoodItem = 'Unknown'
WHERE FoodItem IS NULL;

UPDATE food_data
SET Quantity = 0
WHERE Quantity IS NULL;

UPDATE food_data
SET Price = 0.0
WHERE Price IS NULL;

UPDATE food_data
SET OrderDate = '2025-01-01'
WHERE OrderDate IS NULL;

UPDATE food_data
SET DeliveryTime = '0 mins'
WHERE DeliveryTime IS NULL;

UPDATE food_data
SET PaymentMethod = 'Unknown'
WHERE PaymentMethod IS NULL;

UPDATE food_data
SET Rating = 0
WHERE Rating IS NULL;

select * from food_data;


#Write a query to find the total number of records in a table.
 select count( customername) from food_data; 

select customername from food_data
where customername is  null;

SELECT customername
FROM food_data
WHERE customername IS NULL or customername  = '';

UPDATE food_data
SET customername = NULL
WHERE customername = '';

select count(*) from food_data
where customername is null;

UPDATE food_data
SET rating = NULL
WHERE  rating= '';


select * from food_data;


#Write a query to find the total number of records in a table.
 select count( customername) from food_data;

#Show all distinct restaurant names.
select distinct(restaurantname) 
from food_data
where price is not null;
use food;

#Find the top 5 most expensive food items (based on Price)
SELECT restaurantname, Price 
FROM  food_data
WHERE Price IS NOT NULL
ORDER BY Price DESC
LIMIT 5;

#Count how many orders were placed for each payment method.
select paymentmethod,count(*) as total_payment 
from food_data
group by paymentmethod;

#Count how many orders were placed for each payment method.
select paymentmethod,
count(orderdate) as toatl_orders
from food_data 
group by paymentmethod;

#List all records where the Price is greater than the average price.
select * from food_data
where price> (select avg(price)  from food_data);

#Find the average rating per restaurant.
select restaurantname,avg(rating) as avg_rating
from food_data
group by restaurantname
order by avg_rating desc;

#Find the maximum and minimum delivery time.

select max(deliverytime) as max_deliverytime,
min(deliverytime) as min_deliverytime
from food_data;

#Retrieve all orders placed between '2025-01-01' and '2025-03-01'.
select * from food_data
where orderdate between '2025-01-01' and '2025-03-01';


#Sort all orders in descending order of Price.
select * from food_data
order by price desc;


#Intermediate SQL Questions
#Find all orders without customer names (missing values).
select orderid,orderdate,customername
from food_data
where customername = 'unknown';


select * from food_data;

#Retrieve the 2nd highest order price
SELECT MAX(price) AS second_highest_price
FROM food_data
WHERE price < (SELECT MAX(price) FROM food_data);

#Find total revenue per month from orders.
SELECT 
DATE_FORMAT(orderdate, '%Y-%m') AS month,
SUM(price) AS total_revenue
FROM food_data 
GROUP BY DATE_FORMAT(orderdate, '%Y-%m')
ORDER BY month;

#Write a query to find duplicate customer names.
SELECT customername, COUNT(*) AS name_count
FROM food_data
GROUP BY customername
HAVING COUNT(*) > 1;

#Update the rating to 5 where RestaurantName = 'Dominos'.
update food_data
set rating='5.0'
where RestaurantName ='dominous';

select* from food_data;

#Find the percentage contribution of each restaurant to total sales.
SELECT 
RestaurantName,
SUM(Price) AS TotalSales,
ROUND((SUM(Price) / (SELECT SUM(Price) FROM food_data) * 100), 2) AS PercentageContribution
FROM food_data
GROUP BY RestaurantName;

 #Find the number of unique customers who made purchases.
SELECT COUNT(DISTINCT Customername) AS UniqueCustomerCount
FROM food_data;

# find customers who spent more than the average amount.
SELECT Customername, SUM(Price) AS TotalSpent
FROM food_data
GROUP BY Customername
HAVING SUM(Price) > (
SELECT AVG(CustomerTotal)
FROM (
SELECT SUM(Price) AS CustomerTotal
FROM food_data
GROUP BY Customername
    ) AS Sub
);

#Display the first 10 orders.
SELECT *
FROM food_data
LIMIT 10;

#advanced topics
#Write a query to find the running total of sales per restaurant.
SELECT 
RestaurantName,
OrderDate,
Price,
SUM(Price) OVER (PARTITION BY RestaurantName ORDER BY OrderDate) AS RunningTotal
FROM food_data
ORDER BY RestaurantName, OrderDate;

#Rank orders by Price (highest to lowest).
SELECT 
OrderID,
RestaurantName,
Price,
RANK() OVER (ORDER BY Price DESC) AS PriceRank
FROM food_data
ORDER BY Price DESC;

#Use a CTE to find the top 3 most ordered food items per restaurant.
WITH FoodRank AS (
    SELECT
        RestaurantName,
        FoodItem,
        SUM(Quantity) AS TotalOrdered,
        ROW_NUMBER() OVER (
            PARTITION BY RestaurantName
            ORDER BY SUM(Quantity) DESC
	) AS RankPerRestaurant
FROM food_data
GROUP BY RestaurantName, FoodItem
)
SELECT
    RestaurantName,
    FoodItem,
    TotalOrdered
FROM FoodRank
WHERE RankPerRestaurant <= 3
ORDER BY RestaurantName, TotalOrdered DESC;

#Write a recursive CTE to generate order IDs from 1 to 10
WITH RECURSIVE OrderIDs AS (
    -- Anchor member: start at 1
    SELECT 1 AS OrderID
    UNION ALL
    -- Recursive member: add 1 each time
    SELECT OrderID + 1
    FROM OrderIDs
    WHERE OrderID < 10
)
SELECT OrderID
FROM OrderIDs;

#Create a stored procedure that takes a CustomerName and returns all their orders.
DELIMITER //

CREATE PROCEDURE GetCustomerOrders(IN custName VARCHAR(100))
BEGIN
    SELECT *
    FROM Orders
    WHERE CustomerName = custName;
END //
		
DELIMITER ;

#Create a trigger that updates Quantity in stock after an order is placed.
#Use a CASE WHEN to categorize orders into High (>500), Medium (200–500), Low (<200) based on Price.

SELECT OrderID, CustomerName, Price,
       CASE
           WHEN Price > 500 THEN 'High'
           WHEN Price BETWEEN 200 AND 500 THEN 'Medium'
           ELSE 'Low'
       END AS OrderCategory
FROM food_data;



#Write a query to pivot PaymentMethod into columns showing total sales per method
SELECT 
    RestaurantName,
    SUM(CASE WHEN PaymentMethod = 'Cash' THEN Price ELSE 0 END) AS CashSales,
    SUM(CASE WHEN PaymentMethod = 'Card' THEN Price ELSE 0 END) AS CardSales,
    SUM(CASE WHEN PaymentMethod = 'UPI' THEN Price ELSE 0 END) AS UPISales
FROM food_data
GROUP BY RestaurantName;


#Optimize a query to find orders by restaurant name using an index
-- Create an index on RestaurantName
CREATE INDEX idx_restaurant ON Orders(RestaurantName);

-- Query using the index
SELECT *
FROM food_data
WHERE RestaurantName = 'Dominos';

#Create a view that shows CustomerName, RestaurantName, FoodItem, Price, Rating.
CREATE VIEW CustomerOrderView AS
SELECT CustomerName, RestaurantName, FoodItem, Price, Rating
FROM food_data;

SELECT * FROM CustomerOrderView;


CREATE TABLE food_order (
    order_id INT PRIMARY KEY,
    customer_id INT,
    restaurant_name VARCHAR(255),
    cuisine_type VARCHAR(100),
    cost_of_the_order DECIMAL(10, 2),
    day_of_the_week VARCHAR(20),
    rating VARCHAR(20),
    food_preparation_time INT,
    delivery_time INT
);

CREATE TABLE delivery_alert (
    alert_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT,
    message VARCHAR(255),
    alert_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


DELIMITER //

CREATE TRIGGER check_delivery_time
AFTER INSERT ON food_order
FOR EACH ROW
BEGIN
    IF NEW.delivery_time > 30 THEN
        INSERT INTO delivery_alert (order_id, message)
        VALUES (NEW.order_id, 'Delivery time exceeded 30 minutes');
    END IF;
END;
//

DELIMITER ;

INSERT INTO food_order VALUES
(999999, 12345, 'Test Restaurant', 'Test Cuisine', 50.00, 'Weekday', '4', 25, 35);

SELECT * FROM delivery_alert;

use food;

SELECT 
    RestaurantName,
    ROUND(AVG(Rating), 2) AS Average_Rating,
    COUNT(*) AS Total_Reviews
FROM food_data
GROUP BY RestaurantName
ORDER BY Average_Rating DESC
LIMIT 5;


select * from food_data;
SELECT 
    RestaurantName,
    SUM(Quantity * Price) AS TotalRevenue
FROM food_data
GROUP BY RestaurantName
ORDER BY TotalRevenue DESC;
