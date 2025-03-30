-- =======================================================
-- 📦 Inventory Management System (Pure SQL)
-- =======================================================
-- A professional, advanced SQL-based inventory management system
-- with stock tracking, sales monitoring, stored procedures, and triggers.

-- =======================================================
-- 🛠 1️⃣ DATABASE SCHEMA (TABLES)
-- =======================================================

CREATE TABLE Suppliers (
    SupplierID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Contact VARCHAR(20),
    Email VARCHAR(100),
    Address TEXT
);

CREATE TABLE Products (
    ProductID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Category VARCHAR(50),
    Price DECIMAL(10,2) NOT NULL,
    Quantity INT NOT NULL CHECK (Quantity >= 0),
    SupplierID INT,
    FOREIGN KEY (SupplierID) REFERENCES Suppliers(SupplierID)
);

CREATE TABLE StockTransactions (
    TransactionID INT PRIMARY KEY AUTO_INCREMENT,
    ProductID INT,
    ChangeInQuantity INT NOT NULL,
    TransactionDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

CREATE TABLE Sales (
    SaleID INT PRIMARY KEY AUTO_INCREMENT,
    ProductID INT,
    QuantitySold INT NOT NULL CHECK (QuantitySold > 0),
    TotalPrice DECIMAL(10,2),
    SaleDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

-- =======================================================
-- 📌 2️⃣ INSERT SAMPLE DATA
-- =======================================================

INSERT INTO Suppliers (Name, Contact, Email, Address) VALUES
('Tech Supplies Co.', '123-456-7890', 'techsupplies@example.com', '123 Tech Street, NY'),
('Office Essentials', '987-654-3210', 'officeessentials@example.com', '456 Office Road, CA');

INSERT INTO Products (Name, Category, Price, Quantity, SupplierID) VALUES
('Laptop', 'Electronics', 1200.00, 10, 1),
('Mouse', 'Electronics', 20.00, 50, 1),
('Desk Chair', 'Furniture', 150.00, 15, 2),
('Notebook', 'Stationery', 5.00, 100, 2);

INSERT INTO StockTransactions (ProductID, ChangeInQuantity, TransactionDate) VALUES
(1, 10, '2024-03-01 10:00:00'),
(2, 50, '2024-03-02 11:30:00'),
(3, 15, '2024-03-03 14:00:00'),
(4, 100, '2024-03-04 09:00:00');

INSERT INTO Sales (ProductID, QuantitySold, TotalPrice, SaleDate) VALUES
(1, 2, 2400.00, '2024-03-05 15:30:00'),
(2, 5, 100.00, '2024-03-06 12:00:00'),
(3, 1, 150.00, '2024-03-07 14:45:00'),
(4, 10, 50.00, '2024-03-08 16:00:00');

-- =======================================================
-- 🔄 3️⃣ STORED PROCEDURES (AUTOMATION)
-- =======================================================

DELIMITER //
CREATE PROCEDURE UpdateStockAfterSale(IN p_ProductID INT, IN p_QuantitySold INT)
BEGIN
    UPDATE Products 
    SET Quantity = Quantity - p_QuantitySold 
    WHERE ProductID = p_ProductID;

    INSERT INTO StockTransactions (ProductID, ChangeInQuantity)
    VALUES (p_ProductID, -p_QuantitySold);
END;
//
DELIMITER ;

-- =======================================================
-- 🚀 4️⃣ TRIGGERS (AUTOMATED ACTIONS)
-- =======================================================

DELIMITER //
CREATE TRIGGER PreventNegativeStock 
BEFORE UPDATE ON Products
FOR EACH ROW 
BEGIN
    IF NEW.Quantity < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Error: Not enough stock available!';
    END IF;
END;
//
DELIMITER ;

DELIMITER //
CREATE TRIGGER UpdateStockAfterSaleTrigger
AFTER INSERT ON Sales
FOR EACH ROW
BEGIN
    INSERT INTO StockTransactions (ProductID, ChangeInQuantity)
    VALUES (NEW.ProductID, -NEW.QuantitySold);
END;
//
DELIMITER ;

DELIMITER //
CREATE TRIGGER PreventOutOfStockSales
BEFORE INSERT ON Sales
FOR EACH ROW
BEGIN
    DECLARE current_stock INT;
    SELECT Quantity INTO current_stock FROM Products WHERE ProductID = NEW.ProductID;
    
    IF current_stock < NEW.QuantitySold THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Error: Not enough stock available!';
    END IF;
END;
//
DELIMITER ;

-- =======================================================
-- 📊 5️⃣ USEFUL QUERIES (FOR REPORTS)
-- =======================================================

-- View Inventory Status
SELECT ProductID, Name, Category, Quantity FROM Products ORDER BY Quantity ASC;

-- Total Revenue from Sales
SELECT SUM(TotalPrice) AS TotalRevenue FROM Sales;

-- Top-Selling Products
SELECT P.Name, SUM(S.QuantitySold) AS TotalSold 
FROM Sales S 
JOIN Products P ON S.ProductID = P.ProductID
GROUP BY P.Name 
ORDER BY TotalSold DESC LIMIT 5;

-- List Low Stock Products (Threshold: 5 units)
SELECT * FROM Products WHERE Quantity < 5;

