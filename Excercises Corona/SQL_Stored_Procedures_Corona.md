# Exercise 1

- Write a stored procedure to update already existing values.
- For new_cases for a specific country and report_date.

## (1) If there isn't a record for the specific country and report_date (so you can't do the update), an exception is thrown.

## (2) Check if the new value is reasonable.

- A reasonable value is a value that is not 25% higher or lower than the average number of new cases of the past week.
- If not, throw an exception.

## (3) Do the update.

## (4) Write testcode. Use transactions in order to keep the original data unchanged.

- You can find the error messages in Messages.

## (4.1) Update the new_cases of Belgica on 2021-09-10 to 2200 (previous value was 2190).

## (4.2) Update the new_cases of Belgium on 2021-09-10 to 3200 (previous value was 2190).

## (4.3) Update the new_cases of Belgium on 2021-09-10 to 2200 (previous value was 2190).

### Stored Procedure Code

```sql
CREATE OR ALTER PROCEDURE UpdateNewCases (@country NVARCHAR(50), @report_date datetime2(7), @new_cases_update int)
AS
BEGIN

IF NOT EXISTS (SELECT * FROM CovidData cd JOIN countries c ON c.iso_code = cd.iso_code WHERE country = @country AND report_date = @report_date)
-- If there isn't a record for the specific location and date (so you can't do the update), an exception is thrown
BEGIN
	RAISERROR('There is no such record', 1, 1)
END

DECLARE @avg_new_cases DECIMAL
SELECT  @avg_new_cases = AVG(new_cases) FROM CovidData cd JOIN countries c ON c.iso_code = cd.iso_code WHERE country = @country AND DATEDIFF(DAY, report_date, @report_date) BETWEEN 0 AND 6

IF @new_cases_update NOT BETWEEN (@avg_new_cases * 0.75) AND (@avg_new_cases * 1.25)
	BEGIN
		RAISERROR ('Value not reasonable', 1, 1)
	END
ELSE
	DECLARE @old_new_cases int
	SELECT @old_new_cases = new_cases FROM CovidData cd JOIN countries c ON c.iso_code = cd.iso_code WHERE country = @country AND report_date = @report_date
	DECLARE @diff_cases int
	SET @diff_cases = @new_cases_update - @old_new_cases

	UPDATE CovidData SET new_cases = @new_cases_update WHERE report_date = @report_date AND iso_code =
	(SELECT iso_code FROM countries WHERE country = @country)

END

```

### Testcode 1

```sql
BEGIN TRANSACTION
-- Original value = 2190
EXEC UpdateNewCases 'Belgica', '2021-09-10', 2200

-- Only you (in your session) can see changes
SELECT \* FROM coviddata cd JOIN countries c ON c.iso_code = cd.iso_code
WHERE c.country = 'Belgium' AND report_date >= '2021-09-10'

ROLLBACK;
```

### Testcode 2

```sql
BEGIN TRANSACTION
-- Original value = 2190
EXEC UpdateNewCases 'Belgium', '2021-09-10', 3200

-- only you (in your session) can see changes
SELECT \* FROM coviddata cd JOIN countries c ON c.iso_code = cd.iso_code
WHERE c.country = 'Belgium' AND report_date >= '2021-09-10'

ROLLBACK;
```

### Testcode 3

```sql
BEGIN TRANSACTION
-- Original value = 2190
EXEC UpdateNewCases 'Belgium', '2021-09-10', 2200

-- only you (in your session) can see changes
SELECT \* FROM coviddata cd JOIN countries c ON c.iso_code = cd.iso_code
WHERE c.country = 'Belgium' AND report_date >= '2021-09-10'

ROLLBACK;
```

# Exercise 2

- Write a stored procedure UpdatePopulation to update the population of a country.

## (1) If there isn't a country with the given name, an exception is thrown.

## (2) According to this website: https://www.indexmundi.com/map/?v=24&l=nl the population growth per country is a value between 5% and -8%.

- If the new value for the population for the given country doesn't meet.
- This constraints, an exception is thrown.

## (3) Write testcode. Use transactions in order to keep the original data unchanged.

- You can find the error messages in Messages

## (3.1) Update the population of Belgica to 11600000.

## (3.2) Update the population of Belgium to 21600000.

## (3.3) Update the population of Belgium to 11600000.

### Stored Procedure Code

```sql
CREATE OR ALTER PROCEDURE UpdatePopulation (@country NVARCHAR(50), @new_population int)
AS
BEGIN
IF NOT EXISTS (SELECT * FROM Countries WHERE country = @country)
BEGIN
	RAISERROR('There is no such country', 1, 1)
END

DECLARE @population int
SELECT  @population = population FROM countries WHERE country = @country

IF @new_population NOT BETWEEN (@population * (1-0.08)) AND (@population * (1 + 0.05))
	BEGIN
		RAISERROR ('Value for new population probably not correct', 1, 1)
	END
ELSE
	BEGIN
		UPDATE countries SET population = @new_population WHERE country = @country
	END
END

```

### Testcode 1

```sql
BEGIN TRANSACTION 
-- Original value = 11589616
EXEC UpdatePopulation 'Belgica', 11600000

-- only you (in your session) can see changes
SELECTION population FROM countries WHERE country = 'Belgium'
ROLLBACK;
```

### Testcode 2

```sql
BEGIN TRANSACTION 
-- Original value = 11589616
EXEC UpdatePopulation 'Belgium', 21600000

-- only you (in your session) can see changes
SELECT population FROM countries WHERE country = 'Belgium'

ROLLBACK;
```

### Testcode 3

```sql
BEGIN TRANSACTION
-- Original value = 11589616
EXEC UpdatePopulation 'Belgium', 11600000

-- only you (in your session) can see changes
SELECT population FROM countries WHERE country = 'Belgium'

ROLLBACK;
```

# Exercise 3

- Write a Stored Procedure.

## (1) Calculate and print the startdate of the first golf in Belgium.

## (2) Calculate and print the enddate of the first golf in Belgium.

## (3) Calculate and print the startdate of the second golf in Belgium.

## (4) Calculate and print the enddate of the second golf in Belgium.

## (5) Calculate and print the total number of days and the total number of deaths during the first golf in Belgium.

## (6) Calculate and print the total number of days and the total number of deaths during the second golf in Belgium.

- We define the beginning (ending) of a golf.
- When the 14 days moving average of positive_rate becomes >= (<) 0.06.

```sql
CREATE OR ALTER PROCEDURE Golfs
AS
BEGIN

declare @start_golf1 datetime2(7);

WITH cte
AS
(SELECT report_date, AVG(positive_rate) OVER (ORDER BY report_date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) AS positive_rate_14_days_moving_average
FROM coviddata cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE c.country = 'Belgium')

SELECT @start_golf1 = MIN(report_date)
FROM cte
WHERE positive_rate_14_days_moving_average >= 0.06;

print('Start Golf 1: ' + FORMAT(@start_golf1, 'dd MMM yyyy'));

declare @end_golf1 datetime2(7);

WITH cte
AS
(SELECT report_date, AVG(positive_rate) OVER (ORDER BY report_date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) AS positive_rate_14_days_moving_average
FROM coviddata cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE c.country = 'Belgium')

SELECT @end_golf1 = MIN(report_date)
FROM cte
WHERE positive_rate_14_days_moving_average < 0.06 AND report_date > @start_golf1;

print('End Golf 1: ' + FORMAT(@end_golf1, 'dd MMM yyyy'));



declare @start_golf2 datetime2(7);

WITH cte
AS
(SELECT report_date, AVG(positive_rate) OVER (ORDER BY report_date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) AS positive_rate_14_days_moving_average
FROM coviddata cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE c.country = 'Belgium')

SELECT @start_golf2 = MIN(report_date)
FROM cte
WHERE positive_rate_14_days_moving_average >= 0.06 AND report_date > @end_golf1;

print('Start Golf 2: ' + FORMAT(@start_golf2, 'dd MMM yyyy'));

declare @end_golf2 datetime2(7);

WITH cte
AS
(SELECT report_date, AVG(positive_rate) OVER (ORDER BY report_date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) AS positive_rate_14_days_moving_average
FROM coviddata cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE c.country = 'Belgium')

SELECT @end_golf2 = MIN(report_date)
FROM cte
WHERE positive_rate_14_days_moving_average < 0.06 AND report_date > @start_golf2;

print('End Golf 2: ' + FORMAT(@end_golf2, 'dd MMM yyyy'));

-- Number of days in golf 1 and 2?
DECLARE @number_of_days_golf1 int
SELECT @number_of_days_golf1 = DATEDIFF(day, @start_golf1, @end_golf1);

print('Number of days of Golf 1: ' + CONVERT(VARCHAR, @number_of_days_golf1));

DECLARE @number_of_days_golf2 int
SELECT @number_of_days_golf2 = DATEDIFF(day, @start_golf2, @end_golf2);

print 'Number of days of Golf 2: ' + CONVERT(VARCHAR, @number_of_days_golf2);

-- Number of deaths in golf 1 and 2?
DECLARE @number_of_deaths_golf1 int;
SELECT @number_of_deaths_golf1 = SUM(new_deaths) FROM CovidDATA WHERE report_date BETWEEN @start_golf1 AND @end_golf1 AND country = 'Belgium';
print 'Number of deaths of Golf 1: ' + CONVERT(VARCHAR, @number_of_deaths_golf1);

declare @number_of_deaths_golf2 int;
SELECT @number_of_deaths_golf2 = SUM(new_deaths) FROM CovidDATA WHERE report_date BETWEEN @start_golf2 AND @end_golf2 AND country = 'Belgium';
print 'Number of deaths of Golf 2: ' + CONVERT(VARCHAR, @number_of_deaths_golf2);

END


```

### Testcode

```sql
EXEC Golfs

```

```
Start Golf 1: 07 Mar 2020
End Golf 1: 05 May 2020

Start Golf 2: 05 Oct 2020
End Golf 2: 14 Jan 2021

Number of days in Golf 1: 59
Number of days in Golf 2: 101

Number of deaths in Golf 1: 8016
Number of deaths in Golf 2: 10230
```
