## Example 1

```sql
SELECT \* FROM Employee1

SELECT \* FROM Employee2
```

## Example 2

```sql
SELECT lastname FROM Employee1

SELECT lastname FROM Employee2
```

## Covering index

```sql
SELECT lastname FROM Employee1 WHERE lastname = 'Duffy'

SELECT lastname, title FROM Employee1 WHERE lastname = 'Duffy'

CREATE NONCLUSTERED INDEX EmpLastName_Incl_Title
ON Employee1(lastname) INCLUDE (title);
```

## 1 index with several columns vs. several indexes with 1 column

```sql
CREATE NONCLUSTERED INDEX EmpLastNameFirstname ON Employee1(lastname, firstname);

SELECT lastname, firstname FROM Employee1
WHERE firstname = 'Chris';

```

### Test: Only combined index on lastname and firstname

```sql
DROP INDEX EmpLastName ON Employee1;

SELECT lastname, firstname FROM Employee1
WHERE lastname = 'Preston'

SELECT lastname, firstname FROM Employee1
WHERE firstname = 'Chris';
```

## With extra index on firstname and covering of lastname

```sql
create nonclustered index EmpFirstnameIncLastname ON employee1(firstname)
INCLUDE (lastname);

SELECT lastname, firstname FROM Employee1
WHERE lastname = 'Preston'

SELECT lastname, firstname FROM Employee1
WHERE firstname = 'Chris';
```

## Use of indexes with functions and wildcards

```sql
SELECT lastname, firstname FROM Employee1
WHERE lastname = 'Preston'

SELECT lastname, firstname FROM Employee1
WHERE SUBSTRING(lastname, 2, 1) = 'r'

SELECT lastname, firstname FROM Employee1
WHERE lastname LIKE '%r%'
```

# Tips & Tricks:

## (1) Avoid the use of functions

### BAD

```sql
SELECT lastname, firstname, birthdate
FROM Employee1
WHERE Year(BirthDate) = 1980;
```

### GOOD

```sql
SELECT lastname, firstname, birthdate
FROM Employee1
WHERE BirthDate >= '1980-01-01' AND BirthDate < '1981-01-01';
```

## (2) Avoid the use of functions

```sql
CREATE INDEX EmpLastName ON Employee1 (LastName ASC);
```

### BAD

```sql
SELECT LastName
FROM Employee1
WHERE substring(LastName,1,1) = 'D';
```

### GOOD

```sql
SELECT LastName
FROM Employee1
WHERE LastName like 'D%';
```

## (3) avoid calculations, isolate columns

### BAD

```sql
SELECT EmployeeID, FirstName, LastName
FROM Employee1
WHERE Salary\*1.10 > 100000;

```

### GOOD

```sql
SELECT EmployeeID, FirstName, LastName
FROM Employee1
WHERE Salary > 100000/1.10;
```

## (4) prefer OUTER JOIN above UNION

### BAD

```sql
SELECT lastname, firstname, orderid
from Employee1 e join Orders o on e.EmployeeID = o.employeeid
union
select lastname, firstname, null
from Employee1
where EmployeeID not in (select EmployeeID from Orders)
```

### GOOD

```sql
SELECT lastname, firstname, orderid
from Employee1 e left join Orders o on e.EmployeeID = o.employeeid;
```

## (5) avoid ANY and ALL

### BAD

```sql
SELECT lastname, firstname, birthdate
from Employee1
where BirthDate >= all(select BirthDate from Employee1)
```

### GOOD

```sql
SELECT lastname, firstname, birthdate
from Employee1
where BirthDate = (select max(BirthDate) from Employee1)
```
