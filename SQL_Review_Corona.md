## How many countries can be found in the dataset?

```sql
SELECT COUNT(country) AS Total_Amount_Of_Countries FROM Countries;
```

- **223**

## Give the total population per continent

```sql
SELECT continent , SUM(CONVERT(bigint, population)) AS TotalPopulation FROM Countries
GROUP BY continent;
```

```
| continent     | TotalPopulation |
| :------------ | :-------------- |
| Africa        | 1371693397.0    |
| Asia          | 4652616087.0    |
| Europe        | 749017998.0     |
| North America | 592834824.0     |
| Oceania       | 43198762.0      |
| South America | 433953687.0     |
```

## Which country with more than 1 000 000 inhabitants, has the highest life expectancy?

```sql
SELECT TOP 1 country, life_expectancy
FROM Countries
WHERE population > 1000000
ORDER BY life_expectancy DESC;
```

- Hong Kong

## Calculate the average life_expectancy for each continent, take into account the population for each country.

```sql
SELECT continent, SUM(population * life_expectancy * 1.0)/SUM(population * 1.0) as 'Average_Life_Expectancy'
FROM Countries
GROUP BY continent;
```

| continent Average life expectancy |
| :-------------------------------- |
| Africa 63,5113609481274           |
| Asia 73,4205962661497             |
| Europe 78,901189239207            |
| North America 77,5262297006021    |
| Oceania 78,4352067462026          |
| South America 76,033448358465     |

## Give the country with the highest number of Corona deaths

```sql
SELECT country, SUM(new_deaths) AS TotalDeaths
FROM CovidData cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
GROUP BY country
ORDER BY TotalDeaths DESC
```

### OF

```sql
SELECT country, SUM(new_deaths) AS TotalDeaths
FROM CovidData cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
GROUP BY country
ORDER BY 2 DESC
```

- United States 9002200

## On which day was 50% of the Belgians fully vaccinated?

```sql
SELECT MIN(report_date) AS DagVaccinatiePercent50
FROM CovidData cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE c.country = 'Belgium' AND people_fully_vaccinated >= c.population / 2;
```

- 2021-04-014

## On which day the first Belgian received a vaccin?

```sql
SELECT TOP 1 report_date AS DagEersteVaccinatieOoit
FROM CovidData cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE c.country = 'Belgium' AND people_fully_vaccinated IS NOT NULL
ORDER BY people_fully_vaccinated ASC;
```

- 2020-12-28

## On which day the first Corona death was reported in Europe?

```sql
SELECT TOP 1 report_date AS EersteCovidDodeEU
FROM CovidData cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE c.continent = 'Europe' AND new_deaths IS NOT NULL
ORDER BY report_date ASC;
```

- 2020-01-30

## What is the estimated total amount of smokers in Belgium? Subtract 2 000 000 children from the total Belgian population

```sql
SELECT (female_smokers + male_smokers) / 200 * (population - 2000000) AS TotaleSmokers
FROM Countries
WHERE country = 'Belgium'
```

- 2721134.355

## The first lockdown in Belgium started on 18 march 2020. Give all the data until 21 days afterwards to be able to check if the lockdown had any effect.

```sql
SELECT * FROM CovidData cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE country = 'Belgium' AND report_date BETWEEN '2020/03/18'
AND DATEADD(day, 21,'2020/03/18');
```

## In which month (month + year) the number of deaths was the highest in Belgium?

```sql
Select SUM(new_deaths) as 'Deaths', MONTH(report_date) AS 'Month', YEAR(report_date) AS 'Year'
FROM CovidData cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE country = 'Belgium'
GROUP BY MONTH(report_date), YEAR(report_date)
ORDER BY 1 DESC
```

| Reported year | Reported month | Total number of deaths |
| :------------ | :------------- | :--------------------- |
| 2020          | 4              | 68890                  |
| 2020          | 11             | 50200                  |
| 2020          | 12             | 28830                  |
| 2020          | 5              | 18730                  |
