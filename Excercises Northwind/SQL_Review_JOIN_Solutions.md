## 1. Which suppliers (SupplierID and CompanyName) deliver Dairy Products?

```sql
SELECT DISTINCT s.SupplierID, s.CompanyName
FROM Suppliers s JOIN Products p ON s.SupplierID = p.SupplierID
JOIN Categories c ON p.CategoryID = c.CategoryID
WHERE c.CategoryName LIKE '%Dairy%'
```

| SupplierID | CompanyName                        |
| :--------- | :--------------------------------- |
| 5          | Cooperativa de Quesos 'Las Cabras' |
| 14         | Formaggi Fortini s.r.l.            |
| 15         | Norske Meierier                    |
| 28         | Gai pâturage                       |

## 2. Give for each supplier the number of orders that contain products of that supplier.

- Show supplierID, companyname and the number of orders.
- Order by companyname.

```sql
select s.SupplierID, s.CompanyName, count(DISTINCT od.OrderID) As NrOfOrders
from Suppliers s join Products p ON s.SupplierID = p.SupplierID
JOIN OrderDetails od ON od.ProductID = p.ProductID
GROUP BY s.SupplierID, s.CompanyName
ORDER BY s.CompanyName
```

| SupplierID | CompanyName                        | NrOfOrders |
| :--------- | :--------------------------------- | :--------- |
| 18         | Aux joyeux ecclésiastiques         | 53         |
| 16         | Bigfoot Breweries                  | 64         |
| 5          | Cooperativa de Quesos 'Las Cabras' | 52         |
| 27         | Escargots Nouveaux                 | 18         |
| 1          | Exotic Liquids                     | 90         |
| 29         | Forêts d'érables                   | 71         |

## 3. What’s for each category the lowest UnitPrice? Show category name and unit price.

```sql
SELECT c.CategoryName, MIN(p.UnitPrice) As 'Minimum UnitPrice'
FROM Products p join Categories c ON p.CategoryID = c.CategoryID
GROUP BY c.CategoryName
```

| CategoryName   | Minimum UnitPrice |
| :------------- | :---------------- |
| Beverages      | 4,50              |
| Condiments     | 10,00             |
| Confections    | 9,20              |
| Dairy Products | 2,50              |
| Grains/Cereals | 7,00              |
| Meat/Poultry   | 7,45              |
| Produce        | 10,00             |
| Seafood        | 6,00              |

## 4. Give for each ordered product: productname, the least (columnname 'Min amount ordered') and the most ordered (columnname 'Max amount ordered'). Order by productname.

```sql
SELECT p.ProductName, MIN(od.Quantity) As 'Min amount ordered', Max(od.Quantity) As 'Max amount ordered'
FROM Products p join OrderDetails od ON p.ProductID = od.ProductID
GROUP BY p.ProductName
ORDER BY p.ProductName
```

| ProductName       | Min amount ordered | Max amount ordered |
| :---------------- | :----------------- | :----------------- |
| Alice Mutton      | 2                  | 100                |
| Aniseed Syrup     | 4                  | 60                 |
| Boston Crab Meat  | 1                  | 91                 |
| Camembert Pierrot | 2                  | 100                |
| Carnarvon Tigers  | 4                  | 50                 |
| Chai              | 2                  | 80                 |

## 5. Give a summary for each employee with orderID, employeeID and employeename.

- Make sure that the list also contains employees who don’t have orders yet.

```sql
SELECT e.EmployeeID, e.FirstName + ' ' + e.LastName As 'Name', o.OrderID
FROM Employees e left join Orders o on e.EmployeeID = o.EmployeeID
```

| EmployeeID | Name             | OrderID |
| :--------- | :--------------- | :------ |
| 5          | Steven Buchanan  | 10248   |
| 6          | Michael Suyama   | 10249   |
| 4          | Margaret Peacock | 10250   |
| 3          | Janet Leverling  | 10251   |
| 4          | Margaret Peacock | 10252   |
| 3          | Janet Leverling  | 10253   |
