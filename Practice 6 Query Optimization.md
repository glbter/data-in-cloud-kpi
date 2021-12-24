Завдання:

Використовуючи Azure SQL Database* та [Azure Data Studio](https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver15) (або [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15)) в контексті бази [Northwind](https://github.com/conwid/awesome-northwind):
### 1.	Проаналізувати що виконує наступний запит і переписати його зрозумілим чином:
```sql
SELECT * FROM Orders WHERE ShippedDate = ShippedDate
```

SELECT * FROM Orders WHERE ShippedDate IS NOT NULL


### 2.	Модифікувати запит зменшивши кількість операцій на кожному рядку до однієї:
```sql
UPDATE Products SET Discontinued = (Discontinued - 1) * (-1);
```
```sql
UPDATE Products SET Discontinued = ~Discontinued;
```
### 3.	Перепишіть запит таким чином, щоб зменшити кількість прочитаних рядків до одного:
```sql
SELECT * FROM Employees WHERE CAST(EmployeeID AS NVARCHAR(7)) = N'7'
```
```sql
SELECT * FROM Employees WHERE EmployeeID = 7
```
### 4.	Оптимізувати запит, знаючи, що менша кількість рядків в JOIN покращить результат:
```sql
SELECT CompanyName, MAX(OrderDate) AS LastOrderDate
FROM Orders o 
JOIN Customers c ON o.CustomerID = c.CustomerID
GROUP BY CompanyName
```
```sql
SELECT 
CompanyName, 
o.MaxOrderDate AS LastOrderDate
FROM Customers c
JOIN (
SELECT CustomerId, MAX(OrderDate) AS MaxOrderDate 
FROM Orders 
GROUP BY CustomerId
) AS o 
ON o.CustomerID = c.CustomerID
```
Або викинувши `join`
```sql
SELECT 
	(SELECT CompanyName FROM Customers c WHERE v.CustomerID = c.CustomerID) AS CompanyName,
	LastOrderDate
FROM (
	SELECT CustomerID, MAX(OrderDate) AS LastOrderDate
	FROM Orders o
	GROUP BY [CustomerID]
) v
```
### 5.	Зменшити кількість виразів `SELECT` до одного:
```sql
INSERT INTO [Order Details] (OrderID,ProductID, UnitPrice, Quantity, Discount)
SELECT 11077,
(SELECT ProductID FROM Products WHERE ProductName = 'Carnarvon Tigers'), 
(SELECT UnitPrice FROM Products WHERE ProductName = 'Carnarvon Tigers'),
(SELECT UnitsInStock FROM Products WHERE ProductName = 'Carnarvon Tigers'),
0
```
```sql
SELECT 11077, ProductID, UnitPrice, UnitsInStock, 0 FROM Products WHERE ProductName = 'Carnarvon Tigers'
```

### 6.	Позбавитися повторного читання рядків під час виконання наступного запиту:
```sql
UPDATE Products SET UnitPrice = UnitPrice * 1.05
WHERE UnitPrice > 5
UPDATE Products SET UnitPrice = ROUND(UnitPrice, 2) 
WHERE UnitPrice > 5
```
```sql
UPDATE Products SET UnitPrice = ROUND(UnitPrice * 1.05, 2) 
WHERE UnitPrice > 5
```
### 7.	Після видалення декількох рядків з таблиці запит перестав працювати, необхідно його виправити:
```sql
SELECT * FROM [Order Details] WHERE ProductID = (
SELECT DISTINCT ISNULL(SUM(1), 0) FROM Products WHERE 1=1)
```
```sql
SELECT * 
FROM [Order Details]
WHERE ProductID = (
	SELECT TOP(1) ProductID
	FROM Products
	ORDER BY ProductID DESC
)
```

### 8.	Пришвидшити виконання:
```sql
SELECT * FROM [Order Details] OD
JOIN Products P ON OD.ProductID = P.ProductID
EXCEPT
SELECT * FROM [Order Details] OD
JOIN Products P ON OD.ProductID = P.ProductID AND OD.UnitPrice = P.UnitPrice
```
```sql
SELECT * FROM [Order Details] OD
JOIN Products P ON OD.ProductID = P.ProductID
WHERE OD.UnitPrice != P.UnitPrice
```
### 9.	Знайти спосіб використати індекс:
```sql
SELECT * FROM Products WHERE ProductName = 'Chai'
```
```sql
DROP INDEX [ProductName] ON [dbo].[Products]
CREATE UNIQUE INDEX [ProductName] ON [dbo].[Products] ([ProductName] ASC);
```
### 10.	Оптимізувати швидкодію зберігши результат виконання команди INSERT:
```sql
CREATE TRIGGER TR_OrderInsert ON [Order Details]
INSTEAD OF INSERT
AS
INSERT INTO [Order Details]
SELECT 1
```
```sql
CREATE OR ALTER TRIGGER TR_OrderInsert ON [Order Details]
FOR INSERT
AS
SELECT 1
```
Оскільки триггер заданий у завданні виконує той самий функціонал, не змінюючи insert, краще його взагалі видалити :)
```sql
DROP TRIGGER [Order Details].TR_OrderInsert
```

### Матеріали для повторення:
[Cloud Databases (practice) pt10 (Query optimization (SQL Server Management Studio, Azure Data Studio)](https://www.youtube.com/watch?app=desktop&v=LxYOM3lhazY&feature=youtu.be)  
[Create Unique Indexes](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/create-unique-indexes?view=sql-server-ver15)  
[Query optimization techniques in SQL Server: the basics](https://www.sqlshack.com/query-optimization-techniques-in-sql-server-the-basics/)  
[View Execution Plans in Azure Data Studio](https://www.sqlshack.com/view-execution-plans-in-azure-data-studio/)  

