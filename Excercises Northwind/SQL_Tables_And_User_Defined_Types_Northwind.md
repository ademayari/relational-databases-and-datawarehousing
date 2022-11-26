/************\*\*************/
/\***_ User defined types _**/
/************\*\*************/

/\*

- ~Abstract data types
- Can be used as built-in types
- 2 sorts
  - Distinct types
  - Structured types: tables
- Despite the SQL-2008-standard, many differences between DBMS's
  \*/

/**_ Distinct type _**/
/\*

- Based on a basic type
- Allows to refine existing data types
- The new type is stored in the database
- The distinct type can be used as if it were a built-in type
  \*/

CREATE TYPE IDType FROM INT NOT NULL;
CREATE TYPE NameType FROM NVARCHAR(50) NOT NULL;

/**_ Distinct type: Use _**/
/\*

- Ensure that all person names in the database use the same type
- The distinct type can be used throughout the database
  \*/

CREATE TABLE PERSON
(id   IDType PRIMARY KEY, 
 name NameType);

/**_ Distinct type: DROP _**/
DROP Type IDType;

/\*

- The DROP TYPE statement will not execute when any of the following is true:
  - There are tables in the database that contain columns of the alias data type or the user-defined type
  - There are computed columns, CHECK constraints, schema-bound views
  - There are functions, stored procedures, or triggers created in the database, and these routines use variables or parameters of the alias or user-defined type

\*/

/**_ Distinct type: ALTER _**/
/_
ALTER Type [Name of the type] is not possible.
_/

/************\*\*************/
/**_ Table variables _**/
/************\*\*************/

/**_ Table variables > Local temporary tables _**/
/\*

- Stored in tempdb under "System Databases"
- Only visible:
  - In the creating session
  - At the creating level
  - In all underlying levels
- Removed if creating level goes out-of-scope
  \*/

IF OBJECT_ID('tempdb.dbo.#OrderTotalsByYear') IS NOT NULL  
DROP TABLE dbo.#OrderTotalsByYear;

CREATE TABLE #OrderTotalsByYear
(OrderYear INT NOT NULL PRIMARY KEY,
TotalQuantity INT NOT NULL);

INSERT INTO #OrderTotalsByYear(OrderYear, TotalQuantity)  
SELECT YEAR(o.OrderDate) AS orderyear, SUM(od.Quantity) AS qty
FROM Orders AS o JOIN OrderDetails AS od ON o.orderid = od.orderid
GROUP BY YEAR(orderdate);

SELECT cur.OrderYear, cur.TotalQuantity AS TotalQuantity, prv.TotalQuantity AS TotalQuantityPreviousYear
FROM dbo.#OrderTotalsByYear AS cur LEFT OUTER JOIN dbo.#OrderTotalsByYear AS prv
ON cur.OrderYear = prv.OrderYear + 1;

/**_ Table variables > Global temporary tables _**/
/\*

- Stored in tempdb under "System Databases“
- Visible in all sessions
- Removed if creating session disconnects and there are no more references
  \*/

CREATE TABLE ##ExampleGlobals
(id INT NOT NULL PRIMARY KEY,
username VARCHAR(50) NOT NULL);

INSERT INTO ##ExampleGlobals(id, username)
VALUES(5, 'jjanssens');

SELECT \*  
FROM ##ExampleGlobals;

DROP TABLE ##ExampleGlobals;

/**_ Table variables > @table _**/
DECLARE @OrderTotalsByYear AS TABLE
(OrderYear INT NOT NULL PRIMARY KEY,
TotalQuantity INT NOT NULL);

/**_ Table types _**/
CREATE TYPE TotalOrdersPerYear AS TABLE
(OrderYear INT NOT NULL PRIMARY KEY,
TotalQuantity INT NOT NULL);

DECLARE @TotOrdersPerYear AS TotalOrdersPerYear;

INSERT INTO @TotOrdersPerYear
SELECT YEAR(o.OrderDate) AS orderyear, SUM(od.Quantity) AS qty
FROM Orders AS o JOIN OrderDetails AS od ON o.orderid = od.orderid
GROUP BY YEAR(orderdate);

SELECT \* FROM @TotOrdersPerYear;

/**_ Table types and variables _**/
/\*

- Table types are stored in the DB
- Table variables only exist during batch executions
  (= sequence of statements)
- Advantages of table variables and table types: - Shorter and cleaner code - Table type variables can also be passed as parameters to
  stored procedures and functions
  \*/

/**_ Local temporary tables vs. table variables _**/
/\*

- Both are only visible in creating session
- Table variables have more limited scope: - Only visible in current batch - Not visible in other batches in same session
  \*/

IF OBJECT_ID('tempdb.dbo.#OrderTotalsByYear') IS NOT NULL

DROP TABLE dbo.#OrderTotalsByYear;

CREATE TABLE #OrderTotalsByYear
(OrderYear INT NOT NULL PRIMARY KEY,
TotalQuantity INT NOT NULL);

DECLARE @OrderTotalsByYear AS TABLE
(OrderYear INT NOT NULL PRIMARY KEY,
TotalQuantity INT NOT NULL);

SELECT _ FROM #OrderTotalsByYear
SELECT _ FROM @OrderTotalsByYear

GO -- execute previous commands and start new batch (same as button "Execute" in SSMS)

SELECT _ FROM #OrderTotalsByYear
SELECT _ FROM @OrderTotalsByYear -- Must declare the table variable @OrderTotalsByYear

/**_ Temporary tables: Example _**/
/\*

- We want to delete all orders that contain products from a given supplier
- We first have to delete the orderdetails because of the FK constraint with orders
- But after deleting the orderdetails we loose the link with the supplier and don't know which orders to delete anymore
- Solution: save the orderid's in a temporary table
  \*/

CREATE OR ALTER PROCEDURE DeleteOrdersSupplier @supplierid INT, @nrdeletedorders INT OUTPUT, @nrdeletedorderdetails INT OUTPUT
AS
BEGIN
SET NOCOUNT ON
CREATE TABLE #OrdersSupplier (orderid INT) -- table with 1 column (orderid)

-- insert the OrderID's of the orders with products from the UK that haven't been shipped yet
INSERT INTO #OrdersSupplier
SELECT DISTINCT od.OrderID FROM OrderDetails od JOIN Orders o on od.OrderID = o.OrderID
WHERE ProductID IN
(SELECT ProductID FROM Products WHERE SupplierID = @supplierid);

-- First delete the OrderDetails
DELETE FROM OrderDetails WHERE OrderID IN (SELECT OrderID FROM #OrdersSupplier)
-- How many orderdetails were deleted?
SET @nrdeletedorderdetails = @@rowcount
-- Afterwards delete the Orders
DELETE FROM Orders WHERE OrderID IN (SELECT OrderID FROM #OrdersSupplier)
-- How many orders were deleted?
SET @nrdeletedorders = @@rowcount
END

-- Test of procedure DeleteOrdersSupplier
BEGIN TRANSACTION

-- Choose a value for SupplierID
DECLARE @supplid INT = 1

-- Show which orders should be deleted
SELECT DISTINCT od.OrderID FROM OrderDetails od JOIN Orders o on od.OrderID = o.OrderID
WHERE ProductID IN
(SELECT ProductID FROM Products WHERE SupplierID = @supplid)

DECLARE @nroforders INT
DECLARE @nroforderdetails INT
EXEC DeleteOrdersSupplier @supplid, @nroforders OUT, @nroforderdetails OUT
PRINT 'Nr of deletedorderdetails = ' + str(@nroforderdetails)
PRINT 'Nr of deletedorders = ' + str(@nroforders)

-- Check if the orders are deleted from the database
SELECT DISTINCT od.OrderID FROM OrderDetails od JOIN Orders o on od.OrderID = o.OrderID
WHERE ProductID IN
(SELECT ProductID FROM Products WHERE SupplierID = @supplid)

ROLLBACK
