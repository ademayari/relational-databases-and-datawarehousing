## 1. Count the amount of products (columnname 'amount of products'), AND the amount of products in stock (= unitsinstock not empty) (columnname 'Units in stock').

```sql
SELECT COUNT(ProductID) AS 'Amount of products', SUM(UnitsInStock) AS 'Units in Stock'
FROM Products
```

| Amount of products | Units in Stock |
| :----------------- | :------------- |
| 77                 | 3119           |

## 2. How many employees have a function of Sales Representative (columnname 'Number of Sales Representative')?

```sql
SELECT COUNT(EmployeeID) AS 'Number of Sales Representative'
FROM Employees
WHERE Title = 'Sales Representative'
```

| Number of Sales Representative |
| :----------------------------- |
| 6                              |

## 3. Give the date of birth of the youngest employee (columnname 'Birthdate youngest') and the eldest (columnname 'Birthdate eldest').

```sql
SELECT MAX(BirthDate) AS 'Birthdate Youngest', MIN(BirthDate) AS 'Birthdate Eldest'
FROM Employees
```

| Birthdate Youngest      |
| :---------------------- |
| 1993-08-30 00:00:00.000 |

## 4. What's the number of employees who will retire (at 65) within the first 20 years?

```sql
SELECT COUNT(\*)
FROM Employees
WHERE DATEDIFF(year, Birthdate, GETDATE()) >= 45
```

- 2

## 5. Show a list of different countries where 2 of more suppliers are from. Order alphabeticaly.

```sql
SELECT Country, COUNT(SupplierID) As 'Number of suppliers'
FROM Suppliers
GROUP BY Country
HAVING COUNT(SupplierID) >= 2
```

| Country   | Number of suppliers |
| :-------- | ------------------- |
| Australia | 2                   |
| Canada    | 2                   |
| France    | 3                   |
| Germany   | 3                   |
| Italy     | 2                   |
| Japan     | 2                   |
| Sweden    | 2                   |
| UK        | 2                   |
| USA       | 4                   |

## 6. Which suppliers offer at least 5 products with a price less than 100 dollar? Show supplierId and the number of different products.

```sql
SELECT SupplierID, COUNT(ProductID) As 'Number of products less than 100'
FROM Products
WHERE UnitPrice < 100
GROUP BY SupplierID
HAVING COUNT(ProductID) >= 5

```

- The supplier with the highest number of products comes first.
- | SupplierID | Number of products less than 100 |
  | :--------- | :------------------------------- |
  | 7          | 5                                |
