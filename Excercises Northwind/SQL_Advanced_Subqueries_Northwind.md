# Subqueries in the WHERE OR HAVING clause.

## SUBQUERY that returns a single value

- What is the UnitPrice of the most expensive product?

```sql
SELECT MAX(UnitPrice) AS MaxPrice FROM Products
```

- Alternative?

```sql
SELECT TOP 1 productName, UnitPrice FROM products
ORDER BY UnitPrice DESC
```

- What is the most expensive product?

```sql
SELECT ProductID, ProductName, UnitPrice AS MaxPrice
FROM Products
WHERE UnitPrice = (SELECT MAX(UnitPrice) FROM Products)
```

- Alternative?

```sql
SELECT TOP 1 ProductID, ProductName, UnitPrice AS MaxPrice
FROM Products
ORDER BY UnitPrice DESC
```

- Give the products that cost more than average.

```sql
SELECT ProductID, ProductName, UnitPrice As MaxPrice
FROM Products
WHERE UnitPrice > (SELECT AVG(UnitPrice) FROM Products)
```

- Who is the youngest employee from the USA?

- Give the full name of the employee who earns most

- Give the name of the most frequently sold product

## SUBQUERY that returns a single column

- Give the CustomerID and CompanyName of all customers that already placed an order

```sql
SELECT c.CustomerID, c.CompanyName
FROM Customers c
WHERE c.CustomerID IN (SELECT DISTINCT CustomerID FROM Orders)
```

- Alternative?

```sql
SELECT DISTINCT c.CustomerID, c.CompanyName
FROM Customers c JOIN Orders o ON c.CustomerID = o.CustomerID
```

- Give the CustomerID and CompanyName of all customers that not yet placed an order

```sql
SELECT c.CustomerID, c.CompanyName
FROM Customers c
WHERE c.CustomerID NOT IN (SELECT DISTINCT CustomerID FROM Orders)
```

- Alternative?

```sql
SELECT c.CustomerID, c.CompanyName
FROM Customers c LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE o.CustomerID is NULL
```

- Give the CompanyName of the shippers that haven't shipped anything yet

- Give the productID's for which some customers got a discount and other customers did not

- Give all products that are more expensive than the most expensive product with CategoryName = 'Seafood'

```sql
SELECT \*
FROM Products
WHERE UnitPrice > ALL(SELECT p.UnitPrice
FROM Products p JOIN Categories c ON p.CategoryID = c.CategoryID
WHERE c.CategoryName = 'Seafood')
```

- Alternative?

```sql
SELECT \*
FROM Products
WHERE UnitPrice > (SELECT MAX(UnitPrice) FROM Products p INNER JOIN Categories c ON p.CategoryID = c.CategoryID AND c.CategoryName = 'Seafood')
```

- Give the most expensive product.

```sql
SELECT \*
FROM products
WHERE unitprice >= ALL(SELECT unitprice from products)
```

- Alternative?

```sql
SELECT \*
FROM products
WHERE unitprice = (SELECT MAX(unitprice) from products)
```

- Give all products that are more expensive than one of the products with CategoryName = 'Seafood'

```sql
SELECT \*
FROM Products
WHERE UnitPrice > ANY(SELECT p.UnitPrice FROM Products p INNER JOIN Categories c ON p.CategoryID = c.CategoryID AND c.CategoryName = 'Seafood')
```

- Alternative?

```sql
SELECT \*
FROM Products
WHERE UnitPrice > (SELECT MIN(p.UnitPrice) FROM Products p INNER JOIN Categories c ON p.CategoryID = c.CategoryID AND c.CategoryName = 'Seafood')

```

## Correlated subquerys

```
In a correlated subquery the inner query depends on information from the outer query.
The subquery contains a search condition that refers to the main query,
which makes the subquery depends on the main query
```

- Give employees with a salary larger than the average salary

```sql
SELECT FirstName + ' ' + LastName As FullName, Salary
FROM Employees
WHERE Salary > (SELECT AVG(Salary) FROM Employees)
```

- Give the employees whose salary is larger than the average salary of the employees who report to the same boss.

```sql
SELECT FirstName + ' ' + LastName As FullName, ReportsTo, Salary
FROM Employees As e
WHERE Salary >
(SELECT AVG(Salary) FROM Employees WHERE ReportsTo = e.ReportsTo)
```

- Give all products that cost more than the average unitprice of products of the same category

- Give the customers that ordered more often in 2016 than in 2017

## Subqueries and the EXISTS operator

- Give the CustomerID and CompanyName of all customers that already placed an order

```sql
SELECT c.CustomerID, c.CompanyName
FROM Customers c
WHERE EXISTS
(SELECT \* FROM Orders WHERE CustomerID = c.customerID)
```

- Give the CustomerID and CompanyName of all customers that have not placed any orders yet

```sql
SELECT c.CustomerID, c.CompanyName
FROM Customers c
WHERE NOT EXISTS
(SELECT \* FROM Orders WHERE CustomerID = c.customerID)
```

- Alternative?

```sql
SELECT c.CustomerID, c.CompanyName
FROM Customers c LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE o.CustomerID is NULL

```

- Alternative?

```sql
SELECT CustomerID, CompanyName
FROM Customers
WHERE CustomerID NOT IN (SELECT DISTINCT CustomerID FROM Orders)

```

## Subqueries in the FROM clause

- Give per region the number of orders (region USA + Canada = North America, rest = Rest of World).

### Solution 1

```sql
SELECT
CASE c.Country
WHEN 'USA' THEN 'Northern America'
WHEN 'Canada' THEN 'Northern America'
ELSE 'Rest of world'
END AS Regionclass, COUNT(o.OrderID) As NumberOfOrders
FROM Customers c JOIN Orders o
ON c.CustomerID = o.CustomerID
GROUP BY
CASE c.Country
WHEN 'USA' then 'Northern America'
WHEN 'Canada' then 'Northern America'
ELSE 'Rest of world'
END

```

### Solution 2 -> avoid copy-paste (subquery in FROM)

```sql
SELECT Regionclass, COUNT(OrderID)
FROM
(
SELECT
CASE c.Country
WHEN 'USA' THEN 'Northern America'
WHEN 'Canada' THEN 'Northern America'
ELSE 'Rest of world'
END AS Regionclass, o.OrderID
FROM Customers c JOIN Orders o
ON c.CustomerID = o.CustomerID
)
AS Totals(Regionclass, OrderID)
GROUP BY Regionclass
```

## Subqueries in the SELECT clause

- Give for each employee how much they earn more (or less) than the average salary of all employees with the same supervisor

```sql
SELECT Lastname, Firstname, Salary,
Salary -
(
SELECT AVG(Salary)
FROM Employees
WHERE ReportsTo = e.ReportsTo
)
FROM Employees e

```

## Subqueries in the SELECT and in the FROM clause

- Give per productclass the price of the cheapest product and a product that has that price.

```sql
SELECT Category, MinUnitPrice,
(
SELECT TOP 1 ProductID
FROM Products
WHERE CategoryID = Category AND UnitPrice = MinUnitPrice
) As ProductID
FROM
(
SELECT CategoryID, MIN(UnitPrice)
FROM Products p
GROUP BY CategoryID
) AS CategoryMinPrice(Category, MinUnitPrice);
```

## Application: running totals

- Give the cumulative sum of freight per year

```sql
SELECT OrderID, OrderDate, Freight,
(
SELECT SUM(Freight)
FROM Orders
WHERE YEAR(OrderDate) = YEAR(o.OrderDate) and OrderID <= o.OrderID
) As TotalFreight
FROM Orders o
ORDER BY Orderid;
```

## Exercises

- 1. Give the id and name of the products that have not been purchased yet.
     | Empty dataset |
     | :------------ |

- 2. Select the names of the suppliers who supply products that have not been ordered yet.
     | Empty dataset |
     | :------------ |

- 3. Give a list of all customers from the same country as the customer Maison Dewey
     | CompanyName | country |
     | :--------------- | :------ |
     | Maison Dewey | Belgium |
     | Suprêmes délices | Belgium |

- 4. Calculate how much is earned by the management (like 'president' or 'manager'), the submanagement (like 'coordinator') and the rest
     | TitleClass |TotalSalary |
     | :----------|:---------- |
     | Management |145000.00 |
     | Rest |241000.00 |
     | SubManagment| 51000.00 |

- 5. Give for each product how much the price differs from the average price of all products of the same category
     | ProductID |ProductName| UnitPrice| differenceToCategory |
     | :--------|:-----------|:---------|:-------------------- |
     | 1 Chai |18,00| -19,9791 |
     | 2 Chang |19,00 |-18,9791 |
     | 3 Aniseed Syrup| 10,00| -13,0625 |
     | 4 Chef Anton's Cajun Seasoning| 22,00| -1,0625 |

- 6. Give per title the employee that was last hired
     | title |FullName| HireDate |
     | :-----|:-------|:------------------------------------------------ |
     | Vice President| Sales Andrew Fuller| 2012-08-14 00:00:00.000 |
     | Sales Representative |Anne Dodsworth |2014-11-15 00:00:00.000 |
     | Sales Manager |Steven Buchanan |2013-10-17 00:00:00.000 |
     | Inside Sales Coordinator| Laura Callahan| 2014-03-05 00:00:00.000 |

- 7. Which employee has processed most orders?
     | Margaret Peacock 156 |
     | :------------------- |

- 8. What's the most common ContactTitle in Customers?
     | Owner |
     | :---- |

- 9. Is there a supplier that has the same name as a customer?
     | Empty dataset |
     | :------------ |
