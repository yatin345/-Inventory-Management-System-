# -Inventory-Management-System-
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
