/************\*\*\*\*************/
/** Stored procedures **/
/************\*\*\*\*************/

/\* Introduction
a stored procedure is a named collection of SQL and control-of-flow commands (program) that is stored as a database object

- Analogous to procedures/functions/methods in other languages
- Can be called from a program, trigger or stored procedure
- Is saved in the catalogue
- Accepts input and output parameters
- Returns status information about the correct or incorrect
  execution of the stored procedure
- Contains the tasks to be executed
  \*/

/_ Local variables _/
DECLARE @maxQuantity SMALLINT, @maxUnitPrice MONEY, @minUnitPrice MONEY
SET @maxQuantity = (SELECT MAX(quantity) FROM OrderDetails)
SELECT @maxQuantity = MAX(quantity) FROM OrderDetails
SELECT @maxUnitPrice = MAX(UnitPrice), @minUnitPrice = MIN(UnitPrice) FROM Products

/_ Control flow with Transact SQL_/
CREATE PROCEDURE ShowFirstXEmployees @x INT, @missed INT OUTPUT
AS
DECLARE @empid INT, @fullname VARCHAR(100), @city NVARCHAR(30), @total int

SET @empid = 1
SELECT @total = COUNT(_) FROM Employees
-- SET @total = (SELECT COUNT(_) FROM Employees)
SET @missed = 10

IF @x > @total
SELECT @x = COUNT(\*) FROM Employees
ELSE
SET @missed = @total - @x

WHILE @empid <= @x
BEGIN
SELECT @fullname = firstname + ' ' + lastname, @city = city FROM Employees WHERE employeeid = @empid
PRINT 'Full Name : ' + @fullname
PRINT 'City : '+ @city
PRINT '-----------------------'
SET @empid = @empid + 1
END

-- Testcode
DECLARE @numberOfMissedEmployees INT
SET @numberOfMissedEmployees = 0
EXEC ShowFirstXEmployees 5, @numberOfMissedEmployees OUT -- don't forget OUT
PRINT 'Number of missed employees: ' + STR(@numberOfMissedEmployees)

-- Exercise
-- Write a SP DetailsCategory that gives for a given categoryID
-- (_) the name of this category
-- (_) the number of products in this category --> use SELECT
-- (_) the average price of this category --> use SET
-- The default value for categoryID is 1
-- If there is no category with this categoryID, write an errror message: This category doesn't exist
-- Write Testcode for categoryID = 5 / 10
/_
CategoryName = Grains/Cereals
Number of products = 7
Average Unit price = 20
\*/

-- Changing a SP
ALTER PROCEDURE OrdersSelectAll @customerID int
AS
SELECT \* FROM orders
WHERE customerID = @customerID

-- Removing a SP
DROP PROCEDURE OrdersSelectAll

/_ Return value of SP
-- After execution a SP returns a value
-- (_) Of type int
-- (_) Default return value = 0
-- RETURN statement => Execution of SP stops
-- Return value can be specified
_/

-- Creation of SP with explicit return value
CREATE PROCEDURE OrdersSelectAll AS
SELECT \* FROM Orders
RETURN @@ROWCOUNT

-- Use of SP with return value.
-- Result of print comes in Messages tab.
DECLARE @numberOfOrders INT
EXEC @numberOfOrders = OrdersSelectAll
PRINT 'Number of orders = ' + STR(@numberOfOrders)

/_ SP with parameters
-- A parameter is passed to the SP with an input parameter
-- With an output you can possibly pass a value to the SP and get a value back
-- You can use default values for parameters
_/

CREATE PROCEDURE OrdersSelectAllForCustomer @customerID int = 5, @numberOfOrders int OUTPUT
AS
SELECT @numberOfOrders = COUNT(\*)
FROM orders
WHERE customerID = @customerID

/_ Calling the SP
-- Always provide keyword OUTPUT for output parameters
-- 2 ways to pass actual parameters
(_) use formal parameter name (order unimportant)
(_) positional
_/

-- Pass param by explicit use of formal parameters
DECLARE @nmbrOfOrders INT
EXECUTE OrdersSelectAllForCustomer @customerID = 5, @numberOfOrders = @nmbrOfOrders OUTPUT
PRINT @numberOfOrders

-- Positional parameter passing
DECLARE @nmbrOfOrders INT
EXECUTE OrdersSelectAllForCustomer 5, @nmbrOfOrders OUTPUT
PRINT @numberOfOrders

/************\*\*\*\*************/
/** Error handling **/
/************\*\*\*\*************/

/_
RETURN --> Immediate end of execution of the batch procedure
@@error --> Contains error number of last executed SQL instruction + Value = 0 if OK
RAISERROR --> Returns user defined error or system error
Use of TRY …. CATCH block
_/

/** Error handling --> First approach = using @@error **/

CREATE OR ALTER PROCEDURE ProductInsert @productName nvarchar(50) = NULL, @categoryID int = NULL AS
DECLARE @errormsg int
INSERT INTO Products(ProductName, CategoryID, Discontinued)
VALUES (@productName,@categoryID, 0)

-- save @@error to avoid overwriting by consecutive statements
SET @errormsg = @@error

IF @errormsg = 0
PRINT 'SUCCESS! The product has been added.'
ELSE IF @errormsg = 515
PRINT 'ERROR! ProductName is NULL.'
ELSE IF @errormsg = 547
PRINT 'ERROR! CategoryID doesn''t exist.'
ELSE PRINT 'ERROR! Unable to add new product. Error:' + str(@errormsg)

RETURN @errormsg

-- Testcode
BEGIN TRANSACTION
EXEC ProductInsert 'Wokkels', 10
SELECT \* FROM Products WHERE productName LIKE '%Wokkels%'
ROLLBACK;

/** Error handling --> Second approach = using RAISERROR **/

CREATE OR ALTER PROCEDURE ProductInsert @productName nvarchar(50) = NULL, @categoryID int = NULL AS
DECLARE @errormsg int
INSERT INTO Products(ProductName, CategoryID, Discontinued)
VALUES (@productName,@categoryID, 0)

-- save @@error to avoid overwriting by consecutive statements
SET @errormsg = @@error

IF @errormsg = 0
PRINT 'SUCCESS! The product has been added.'
ELSE IF @errormsg = 515
RAISERROR ('ProductName or CategoryID is NULL.',18,1)
ELSE IF @errormsg = 547
RAISERROR ('CategoryID doesn''t exist.',18,1)
ELSE PRINT 'ERROR! Unable to add new product. Error:' + str(@errormsg)

RETURN @errormsg

-- Testcode
BEGIN TRANSACTION
EXEC ProductInsert 'Wokkels', 10
SELECT \* FROM Products WHERE productName LIKE '%Wokkels%'
ROLLBACK;

/** Exception handling: catch-block functions **/
/_
ERROR_LINE(): line number where exception occurred
ERROR_MESSAGE(): error message
ERROR_PROCEDURE(): SP where exception occurred
ERROR_NUMBER(): error number
ERROR_SEVERITY(): severity level
_/

CREATE OR ALTER PROCEDURE DeleteShipper @ShipperID int = NULL, @NumberOfDeletedShippers int OUT
AS
BEGIN
BEGIN TRY
DELETE FROM Shippers WHERE ShipperID = @ShipperID
SET @NumberOfDeletedShippers = @@ROWCOUNT
END TRY
BEGIN CATCH
PRINT 'Error Number = ' + STR(ERROR_NUMBER());
PRINT 'Error Procedure = ' + ERROR_PROCEDURE();
PRINT 'Error Message = ' + ERROR_MESSAGE();
END CATCH
END

-- Testcode
BEGIN TRANSACTION
DECLARE @nrOfDeletedShippers int;
EXEC DeleteShipper 3, @nrOfDeletedShippers OUT;
PRINT 'Number of deleted shippers ' + STR(@nrOfDeletedShippers);
ROLLBACK

/** Throw **/
/\*

- Is an alternative to RAISERROR
- Without parameters: only in catch block (= rethrowing)
- With parameters: also outside catch block
  \*/

-- Throw: Example 1 - Without parameters => rethrowing the exception

CREATE OR ALTER PROCEDURE DeleteShipper @ShipperID int = NULL, @NumberOfDeletedShippers int OUT
AS
BEGIN
BEGIN TRY
DELETE FROM Shippers WHERE ShipperID = @ShipperID
SET @NumberOfDeletedShippers = @@ROWCOUNT
END TRY
BEGIN CATCH
PRINT 'This is an error';
THROW -- if you put this in comment => the exception is caught and isn't shown in Messages
END CATCH
END

-- Testcode
BEGIN TRANSACTION
DECLARE @nrOfDeletedShippers int;
EXEC DeleteShipper 3, @nrOfDeletedShippers OUT;
PRINT 'Number of deleted shippers ' + STR(@nrOfDeletedShippers);
ROLLBACK

-- Throw: Example 2 - With parameters => create your own user defined exception

CREATE OR ALTER PROCEDURE DeleteShipper @ShipperID int = NULL, @NumberOfDeletedShippers int OUT
AS
BEGIN
BEGIN TRY
DELETE FROM Shippers WHERE ShipperID = @ShipperID
SET @NumberOfDeletedShippers = @@ROWCOUNT
END TRY
BEGIN CATCH
PRINT 'This is an error';
THROW 50001, 'The shipper isn''t deleted', 1
END CATCH
END

-- Testcode
BEGIN TRANSACTION
DECLARE @nrOfDeletedShippers int;
EXEC DeleteShipper 3, @nrOfDeletedShippers OUT;
PRINT 'Number of deleted shippers ' + STR(@nrOfDeletedShippers);
ROLLBACK

-- DeleteShipper revisited

CREATE OR ALTER PROCEDURE DeleteShipper @ShipperID int = NULL, @NumberOfDeletedShippers int OUT
AS
BEGIN
IF @ShipperID IS NULL
BEGIN
PRINT 'Please provide a shipperID'
RETURN
END

IF NOT EXISTS (SELECT \* FROM Shippers where ShipperID = @ShipperID)
BEGIN
PRINT 'The shipper doesn''t exist.'
RETURN
END

IF EXISTS (SELECT \* FROM Orders where ShipVia = @ShipperID)
BEGIN
PRINT 'The shipper already has orders and can''t be deleted.'
RETURN
END

DELETE FROM Shippers WHERE ShipperID = @ShipperID
PRINT 'The shipper has been succesfully deleted'

END

-- Testcode
BEGIN TRANSACTION
DECLARE @nrOfDeletedShippers int;
EXEC DeleteShipper NULL, @nrOfDeletedShippers OUT;
PRINT 'Number of deleted shippers ' + STR(@nrOfDeletedShippers);
ROLLBACK

-- SP Example: Insert Shipper with identity
CREATE OR ALTER PROCEDURE InsertShipper @CompanyName NVARCHAR(40), @phone NVARCHAR(24) = NULL, @shipperID INT OUT
AS
INSERT INTO Shippers(CompanyName, Phone)
VALUES (@CompanyName, @Phone)

SET @shipperID = @@identity

-- Testcode
BEGIN TRANSACTION
DECLARE @NewShipperID INT;
EXEC InsertShipper 'Solid Shippings', '(503) 555-9874', @NewShipperID OUT;
PRINT 'ID of inserted shipper: ' + STR(@NewShipperID);
ROLLBACK

/************\*\*\*\*************/
/** User defined functions **/
/************\*\*\*\*************/

-- Example: Give the age of each employee

-- Step 1
SELECT lastname, firstname, birthdate, GETDATE() As today, DATEDIFF(YEAR,birthdate,GETDATE()) age
FROM employees

-- Step 2
SELECT lastname, firstname, birthdate, GETDATE() As today, DATEDIFF(DAY,birthdate,GETDATE()) / 365 age
FROM employees

-- Step 3

CREATE FUNCTION GetAge (@birthdate AS DATE, @eventdate AS DATE)
RETURNS INT
AS
BEGIN
RETURN
DATEDIFF(year, @birthdate, @eventdate)
 - CASE WHEN 100 _ MONTH(@eventdate) + DAY(@eventdate)  
           < 100 _ MONTH(@birthdate) + DAY(@birthdate) 
THEN 1 ELSE 0
END;
END;

-- How to use?
SELECT lastname, firstname, birthdate, GETDATE() As today, dbo.GetAge(birthdate,GETDATE()) age
FROM employees

-- Exercise: Write a function that calculates the netto salary per month for each employee
-- If salary < 4000 EUR per month => tax is 30%
-- If salary < 5500 EUR per month => tax is 35%
-- Else => tax is 40%

-- Give an overview of firstname, lastname, birthdate, salary and netto salary for each employee

-- Give an overview of all employees that earn more than 2800 each month

/******\*\*\*******/
/** Exercises **/
/******\*\*\*******/

-- Exercise 1
-- In case a supplier goes bankrupt, we need to inform the customers on this
-- Write a SP ContactCustomers that gives all information about the customers that ordered a product of this supplier during the last 6 months,
-- given the companyname of the supplier, so we will be able to inform the appropriate customers
-- First check if the companyname IS NOT NULL and if there is a supplier with this companyname
-- Then check if there isn't by chance more than 1 supplier with this companyname
-- Use in the procedure '2018-10-21' instead of GETDATE(), otherwise the procedure won't return any records.
-- The procedure returns the number of found customers using an OUTPUT parameter

-- Write testcode
-- (_) in which companyName IS NULL
-- (_) in which companyName does not exist
-- (\*) in which companyName = Refrescos Americanas LTDA.

-- Exercise 2
-- We'd like to have 1 stored procedure InsertProduct to insert new OrderDetails, however make sure that:

-- (_) the OrderID and ProductID exist;
-- (_) the UnitPrice is optional, use it when it's given else retrieve the product's price from the product table.
-- If the difference between the given UnitPrice and the UnitPrice in the table Products is larger than 15%: write out a message and don't insert the new OrderDetail
-- (_) If the discount is smaller than 0.0 or larger than 0.25: write out a message and don't insert the new OrderDetail.
-- The discount is optional. If there is no value for discount => the discount is 0.0
-- (_) If the quantity is larger than 2 \* the max ordered quantity of this product untill now: write out a message and don't insert the new OrderDetail

-- Write testcode in which you check all the special cases
-- Put everything in a transaction, so the original data in the tables isn't changed. You can see messages in the Messages tab
-- An example that should work: OrderID = 10249 + ProductID = 72 + UnitPrice = 35.00 + qauntity = 10 + discount = 0.15
-- OrderID = 10249 contains already 2 rows:
-- OrderID ProductID UnitPrice Quantity Discount
-- 10249 14 18,60 9 0
-- 10249 51 42,40 40 0

-- TestCode

BEGIN TRANSACTION

SELECT \* FROM OrderDetails WHERE OrderID = 10249

ROLLBACK;

-- Exercise 3
-- Create a stored procedure for deleting a shipper. You can only delete a shipper if
-- (_) The shipperID exists
-- (_) There are no Orders for this shipper

-- Write two versions of your procedure:
-- (\*) In the first version (DeleteShipper1) you check these conditions before deleting the shipper,

-- so you don't rely on SQL Server messages. Generate an appropriate error message if the shipper can't be deleted.
-- (\*) In the second version (DeleteShipper2) you try to delete the shipper and catch the exceptions that might occur.

-- Write testcode to delete shipper with shipperID = 10 (doesn't exist) / 5 (exists + no shippings) / 3 (exists + already shippings).
-- Put everything in a transaction. Messages are visible on the Messages tab

-- Version 1

-- Testcode
BEGIN TRANSACTION

SELECT \* FROM Shippers

ROLLBACK;

-- Version 2

-- Testcode
BEGIN TRANSACTION

SELECT \* FROM Shippers

ROLLBACK;
