Food Delivery Analysis
** Project Description**

This project simulates a food delivery platform (like Zomato or Swiggy) using SQL. The goal is to design a relational database that tracks customers, restaurants, menu items, and orders.
With this data, you can run analytics such as order frequency, revenue per restaurant, popular items, and payment trends.

** About**

Food delivery apps connect customers with restaurants, allowing them to browse menus, place orders, and pay online.
A backend database stores all interactions — customers, orders, delivery times, payments, and ratings.

**Tables**
Table Name	Purpose
customers	Stores customer details
restaurants	Stores restaurant information
menu_items	Stores items offered by each restaurant (FoodItem, Price)
orders	Tracks each order’s metadata (order date, delivery time, payment, rating)
order_items	Tracks which items and quantities belong to which order

**Data Cleaning**
UPDATE food_data SET CustomerName = 'Unknown' WHERE CustomerName IS NULL;
UPDATE food_data SET RestaurantName = 'Unknown' WHERE RestaurantName IS NULL;
UPDATE food_data SET FoodItem = 'Unknown' WHERE FoodItem IS NULL;
UPDATE food_data SET Quantity = 0 WHERE Quantity IS NULL;
UPDATE food_data SET Price = 0.0 WHERE Price IS NULL;
UPDATE food_data SET OrderDate = '2025-01-01' WHERE OrderDate IS NULL;
UPDATE food_data SET DeliveryTime = '0 mins' WHERE DeliveryTime IS NULL;

-- Total records in the table
SELECT COUNT(*) FROM food_data;

-- Distinct restaurant names
SELECT DISTINCT restaurantname FROM food_data;

-- Top 5 most expensive food items
SELECT restaurantname, Price 
FROM food_data
ORDER BY Price DESC
LIMIT 5;

-- Orders by payment method
SELECT paymentmethod, COUNT(*) AS total_orders
FROM food_data
GROUP BY paymentmethod;

-- Average rating per restaurant
SELECT restaurantname, AVG(rating) AS avg_rating
FROM food_data
GROUP BY restaurantname
ORDER BY avg_rating DESC;

UPDATE food_data SET PaymentMethod = 'Unknown' WHERE PaymentMethod IS NULL;
UPDATE food_data SET Rating = 0 WHERE Rating IS NULL;



-- 2nd highest order price
SELECT MAX(price) AS second_highest_price
FROM food_data
WHERE price < (SELECT MAX(price) FROM food_data);

-- Total revenue per month
SELECT DATE_FORMAT(orderdate, '%Y-%m') AS month,
       SUM(price) AS total_revenue
FROM food_data
GROUP BY DATE_FORMAT(orderdate, '%Y-%m')
ORDER BY month;

-- Percentage contribution of each restaurant to total sales
SELECT RestaurantName,
       SUM(Price) AS TotalSales,
       ROUND((SUM(Price) / (SELECT SUM(Price) FROM food_data) * 100), 2) AS PercentageContribution
FROM food_data
GROUP BY RestaurantName;


-- Running total of sales per restaurant
SELECT RestaurantName, OrderDate, Price,
       SUM(Price) OVER (PARTITION BY RestaurantName ORDER BY OrderDate) AS RunningTotal
FROM food_data;

-- Rank orders by price
SELECT OrderID, RestaurantName, Price,
       RANK() OVER (ORDER BY Price DESC) AS PriceRank
FROM food_data;

-- Recursive CTE to generate order IDs from 1 to 10
WITH RECURSIVE OrderIDs AS (
    SELECT 1 AS OrderID
    UNION ALL
    SELECT OrderID + 1
    FROM OrderIDs
    WHERE OrderID < 10
)
SELECT OrderID
FROM OrderIDs;


-- Trigger to check delivery time
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

-- Stored procedure to get customer orders
DELIMITER //
CREATE PROCEDURE GetCustomerOrders(IN custName VARCHAR(100))
BEGIN
    SELECT * FROM Orders WHERE CustomerName = custName;
END //
DELIMITER ;




