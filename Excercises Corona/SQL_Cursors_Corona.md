# Exercise 1

## Create the following overview. First give the SQL statement, then use a cursor to get the following layout.

```
Africa

- Number of countries = 55
- Total population = 1,371,693,397
  Asia
- Number of countries = 49
- Total population = 4,652,616,087
  Europe
- Number of countries = 50
- Total population = 749,017,998
  North America
- Number of countries = 35
- Total population = 592,834,824
  Oceania
- Number of countries = 21
- Total population = 43,198,762
  South America
- Number of countries = 13
- Total population = 433,953,687
```

### SQL Statement

```sql
SELECT continent, COUNT(DISTINCT country) AS Number_of_countries, SUM(CAST(population AS BIGINT)) AS Total_population
FROM countries
GROUP BY continent
```

### Cursor

```sql
DECLARE @continent varchar(100), @number_of_countries int, @total_population bigint

DECLARE corona_cursor_5 CURSOR FOR
SELECT continent, COUNT(DISTINCT country) AS Number_of_countries, SUM(CAST(population AS BIGINT)) AS Total_population
FROM countries
GROUP BY continent

OPEN corona_cursor_5
FETCH NEXT FROM corona_cursor_5 INTO  @continent, @number_of_countries, @total_population
WHILE @@FETCH_STATUS = 0
	BEGIN
		PRINT @continent
		PRINT ' - Number of countries = ' + str(@number_of_countries)
		PRINT ' - Total population = ' + format(@total_population, 'N0')
		FETCH NEXT FROM corona_cursor_5 INTO  @continent, @number_of_countries, @total_population
	END
CLOSE corona_cursor_5

DEALLOCATE corona_cursor_5
```

# Exercise 2

## Give per continent a list with the 5 countries with the highest number of deaths.

## Step 1: Give a list of all continents. First give the SQL statement, then use a cursor to get the following layout.

```
- Africa
- Asia
- Europe
- North America
- Oceania
- South America
```

```sql
DECLARE @continent varchar(50)

DECLARE cursor_corona_3 CURSOR FOR
SELECT DISTINCT continent
FROM countries

OPEN cursor_corona_3

FETCH NEXT FROM cursor_corona_3 INTO @continent

WHILE @@FETCH_STATUS = 0
BEGIN
PRINT ' - ' + @continent
FETCH NEXT FROM cursor_corona_3 INTO @continent
END

CLOSE cursor_corona_3
DEALLOCATE cursor_corona_3

```

## Step 2: Give the countries with the highest number of deaths for Africa. First give the SQL statement, then use a cursor to get the following layout.

```
South Africa 101823
Tunisia 29247
Egypt 24796
Morocco 16276
Ethiopia 7572
```

```sql
DECLARE @country varchar(50), @new_deaths int

DECLARE cursor_corona_4 CURSOR FOR
SELECT TOP 5 c.country, sum(ISNULL(cd.new_deaths, 0))
FROM countries c JOIN CovidData cd ON c.iso_code = cd.iso_code
WHERE c.continent = 'Africa'
GROUP BY c.country
ORDER BY 2 DESC;

OPEN cursor_corona_4

FETCH NEXT FROM cursor_corona_4 INTO @country, @new_deaths

WHILE @@FETCH_STATUS = 0
BEGIN
PRINT @country + ' ' + str(@new_deaths)
FETCH NEXT FROM cursor_corona_4 INTO @country, @new_deaths
END

CLOSE cursor_corona_4
DEALLOCATE cursor_corona_4
```

## Step 3: Combine both cursors to get the following result.

```
- Africa
  South Africa 101823
  Tunisia 29247
  Egypt 24796
  Morocco 16276
  Ethiopia 7572
- Asia
  India 370118
  Indonesia 157849
  Iran 144258
  Turkey 101068
  Philippines 62455
- Europe
  Russia 377921
  United Kingdom 205707
  Italy 176495
  France 155100
  Germany 148866
- North America
  United States 900220
  Mexico 322723
  Canada 45279
  Guatemala 19701
  Honduras 11083
- Oceania
  Australia 14276
  New Zealand 1979
  Fiji 880
  Papua New Guinea 664
  French Polynesia 649
- South America
  Brazil 634116
  Peru 216938
  Colombia 141746
  Argentina 126479
  Chile 48531
```

## Step 4: Replace the TOP 5 values by a cte with dense_rank.

```sql
DECLARE @continent nvarchar(50)
DECLARE @country nvarchar(50), @new_deaths int
DECLARE cursor_corona_3 CURSOR FOR
SELECT DISTINCT continent
FROM countries

OPEN cursor_corona_3

FETCH NEXT FROM cursor_corona_3 INTO @continent

WHILE @@FETCH_STATUS = 0

BEGIN
PRINT ' - ' + @continent
-- begin inner cursor
DECLARE cursor_corona_4 CURSOR FOR
WITH cte_corona(country, total_deaths, d_rank) AS
(
SELECT TOP 5 c.country, sum(ISNULL(cd.new_deaths, 0)),
dense_rank() OVER (ORDER BY sum(ISNULL(new_deaths, 0)) DESC) AS 'rank'
FROM countries c JOIN CovidData cd ON c.iso_code = cd.iso_code
WHERE c.continent = @continent
GROUP BY c.country
)

SELECT country, total_deaths FROM cte_corona WHERE d_rank <= 5;

OPEN cursor_corona_4

FETCH NEXT FROM cursor_corona_4 INTO @country, @new_deaths

WHILE @@FETCH_STATUS = 0
BEGIN
PRINT @country + ' ' + str(@new_deaths)
FETCH NEXT FROM cursor_corona_4 INTO @country, @new_deaths
END

CLOSE cursor_corona_4
DEALLOCATE cursor_corona_4
-- end inner cursor
FETCH NEXT FROM cursor_corona_3 INTO @continent
END

CLOSE cursor_corona_3
DEALLOCATE cursor_corona_3

SELECT year(report_date), datepart(week, report_date), SUM(new_cases) / 500.0
FROM coviddata cd JOIN countries c ON cd.iso_code = c.iso_code
WHERE year(report_date) = 2021 AND c.country = 'Belgium'
GROUP BY year(report_date), datepart(week, report_date)
```

# Exercise 3

## Make the following, visual overview for the total number of new_cases / 5000 for Belgium for each week starting from 2021-01-01.

- This makes it more clear which are the weeks with a lot of new_cases.
- Use the function REPLICATE to get the x's.

### declare cursor

```sql
DECLARE corona_cursor CURSOR
FOR
SELECT YEAR(report_date), DATEPART(week, report_date), sum(new_cases) / 5000
FROM coviddata cd JOIN countries c ON cd.iso_code = c.iso_code
WHERE c.country = 'Belgium' AND report_date >= '2021-01-01'
GROUP BY YEAR(report_date), DATEPART(week, report_date)
ORDER BY 1, 2

DECLARE @year INT, @weeknumber INT, @sumNewCases INT
```

### open cursor

```sql
OPEN corona_cursor
```

### fetch data

```sql
FETCH NEXT FROM corona_cursor INTO @year, @weeknumber, @sumNewCases

WHILE @@FETCH_STATUS = 0
BEGIN
	PRINT str(@year) + ' ' + str(@weekNumber, 2) + ' ' + REPLICATE('x',@sumNewCases)
  	FETCH NEXT FROM corona_cursor INTO @year, @weeknumber, @sumNewCases
END

```

### close cursor

```sql
CLOSE corona_cursor
```

### deallocate cursor

```sql
DEALLOCATE corona_cursor
```

```
2021 1
2021 2 xx
2021 3 xx
2021 4 xx
2021 5 xxx
2021 6 xxx
2021 7 xx
2021 8 xxx
2021 9 xxx
2021 10 xxx
2021 11 xxx
2021 12 xxxx
2021 13 xxxxxxx
2021 14 xxxxxx
2021 15 xxxxx
2021 16 xxxx
2021 17 xxxxx
2021 18 xxxx
2021 19 xxxx
2021 20 xxx
2021 21 xxx
2021 22 xx
2021 23 xx
2021 24 x
2021 25
2021 26
2021 27
2021 28 x
2021 29 x
2021 30 xx
2021 31 xx
2021 32 xx
2021 33 xx
2021 34 xx
2021 35 xx
2021 36 xx
2021 37 xx
2021 38 xx
2021 39 xx
2021 40 xx
2021 41 xx
2021 42 xxx
2021 43 xxxxxx
2021 44 xxxxxxxxx
2021 45 xxxxxxxxxx
2021 46 xxxxxxxxxxxxxx
2021 47 xxxxxxxxxxxxxxxxxxx
2021 48 xxxxxxxxxxxxxxxxxxxxxxxx
2021 49 xxxxxxxxxxxxxxxxxxxxxxxxx
2021 50 xxxxxxxxxxxxxxxxxxxx
2021 51 xxxxxxxxxxxxx
2021 52 xxxxxxx
2021 53 xxxxxxxxxxxxx
2022 1
2022 2 xxxxxxxxxxxxxxxxxxxxxxxxx
2022 3 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
2022 4 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
2022 5 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
2022 6 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
2022 7 xxxxxxxxxxxxxxxxxxxxxxxxx
2022 8 xxxxxxxxxxxxxx
2022 9 xxxxxxxxx
2022 10 xxxxxxxx
2022 11 xxxxxxxxxx
2022 12 xxxxxxxxxxxxx
2022 13 xxxxxxxxxxxxxx
2022 14 xxxxxxxxxxxxx
2022 15 xxxxxxxxxxxx
2022 16 xxxxxxxxxxx
2022 17 xxxxxxxx
2022 18 xxxxxxxx
2022 19 xxxxxx
2022 20 xxxxx
2022 21 xxx
2022 22 xx
2022 23 xx
2022 24 xx
2022 25 xxx
2022 26 xxxx
2022 27 xxxxxx
2022 28 xxxxxxxxx
2022 29 xxxxxxxxxx
2022 30 xxxxxx
2022 31 xxxxxx
2022 32 xxxx
2022 33 xxx
2022 34 xx
2022 35 xx
2022 36 xx
2022 37 x
2022 38 xx
```
