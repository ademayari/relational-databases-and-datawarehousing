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
continent	TotalPopulation
Africa	1371693397.0
Asia	4652616087.0
Europe	749017998.0
North America	592834824.0
Oceania	43198762.0
South America	433953687.0
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

- continent Averae life expectancy
- Africa 63,5113609481274
- Asia 73,4205962661497
- Europe 78,901189239207
- North America 77,5262297006021
- Oceania 78,4352067462026
- South America 76,033448358465

## Give the country with the highest number of Corona deaths

- United States 9002200

## On which day was 50% of the Belgians fully vaccinated?

- 2021-04-014

## On which day the first Belgian received a vaccin?

- 2020-12-28

## On which day the first Corona death was reported in Europe?

- 2020-01-30

## What is the estimated total amount of smokers in Belgium?

## Subtract 2 000 000 children from the total Belgian population

- 2721134.355

# The first lockdown in Belgium started on 18 march 2020. Give all the data until 21 days afterwards to be able to check if the lockdown had any effect.

# In which month (month + year) the number of deaths was the highest in Belgium?

- Reported year Reported month Total number of deaths
- 2020 4 68890
- 2020 11 50200
- 2020 12 28830
- 2020 5 18730
