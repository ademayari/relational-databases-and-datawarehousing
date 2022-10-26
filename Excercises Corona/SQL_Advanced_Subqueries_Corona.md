## 1. On which day(s) the highest number of new cases was reported in Belgium?

```sql
SELECT report_date, new_cases
FROM CovidData  cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE c.country = 'Belgium' AND new_cases = (SELECT MAX(new_cases) FROM CovidData cd INNER JOIN Countries c ON c.iso_code = cd.iso_code WHERE c.country = 'Belgium')
```

| report_date             | new_cases |
| :---------------------- | :-------- |
| 2022-01-24 00:00:00.000 | 133480    |

## 2. On which day(s) the highest number of new deaths was reported for each country?

```sql
SELECT country, report_date, new_deaths
FROM CovidData  cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE new_deaths = (SELECT MAX(new_deaths) FROM CovidData WHERE country = c.country);
```

| country       | report_date             | new_deaths |
| :------------ | :---------------------- | :--------- |
| United States | 2022-02-10 00 00:00.000 | 3251       |

## 3. Which country(ies) was(were) the first to start vaccinations?

```sql
SELECT country, report_date, total_vaccinations
FROM CovidData  cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE total_vaccinations > 0  AND report_date = (SELECT MIN(report_date) FROM CovidData WHERE total_vaccinations > 0);
```

| country | report_date             |
| :------ | :---------------------- |
| Norway  | 2020-12-02 00:00:00.000 |

## 4. Give for each country the percentage of fully vaccinated people.

- based on the most recent data on the fully vaccinated people for that country.
- Order the results in a descending way.
- You could try to solve this taking into account that (_for now_) the number of fully vaccinated people is an always increasing number.
- But once the vaccination campaign is done and old people are dying and new babies are born, it's possible this won't be the case any more.
-

```sql
SELECT c.country, cd.people_fully_vaccinated * 1.0 / c.population
FROM CovidData  cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE people_fully_vaccinated IS NOT NULL AND report_date = (SELECT MAX(report_date) FROM CovidData WHERE iso_code = cd.iso_code AND people_fully_vaccinated IS NOT NULL)
ORDER BY 2 DESC;
```

| country   | PercentageFullyVaccinated |
| :-------- | :------------------------ |
| Gibraltar | 1.229438128877            |
| Samoa     | 1.056514309697            |
| Brunei    | 1.006522743538            |
| Pitcairn  | 1.000000000000            |

## 5. Assume that all people in Belgium got fully vaccinated from elder to younger. We don't take into account people on priority lists like doctors, nurses, .... On which day all Belgians of 70 or older were fully vaccinated?

```sql
SELECT MIN(report_date)
FROM CovidData  cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
WHERE country = 'Belgium' AND people_fully_vaccinated >= (SELECT aged_70_older * population / 100 FROM Countries WHERE country = 'Belgium');
```

| 2021-05-18 |
| :--------- |

## 6. Give an overview of the cumulative sum of Corona deaths for each country.

- Give country, report_date, new_deaths and the cumulative sum.
- Order by country and report_date.

```sql
SELECT country, report_date, new_deaths,
(
	SELECT SUM(new_deaths)
	FROM CovidData
	WHERE iso_code = cd.iso_code AND report_date <= cd.report_date
) As TotalDeaths
FROM CovidData  cd INNER JOIN Countries c ON c.iso_code = cd.iso_code
ORDER BY country, report_date;
```

| country     | report_date             | new_deaths | TotalDeaths |
| :---------- | :---------------------- | :--------- | :---------- |
| Afghanistan | 2020-03-23 00:00:00.000 | 1          | 1           |
| Afghanistan | 2020-03-24 00:00:00.000 | 0          | 1           |
| Afghanistan | 2020-03-25 00:00:00.000 | 0          | 1           |
| Afghanistan | 2020-03-26 00:00:00.000 | 1          | 2           |
| Afghanistan | 2020-03-27 00:00:00.000 | 0          | 2           |
| Afghanistan | 2020-03-28 00:00:00.000 | 0          | 2           |
| Afghanistan | 2020-03-29 00:00:00.000 | 2          | 4           |
| Afghanistan | 2020-03-30 00:00:00.000 | 0          | 4           |
| Afghanistan | 2020-03-31 00:00:00.000 | 0          | 4           |
| Afghanistan | 2020-04-01 00:00:00.000 | 0          | 4           |

## 7. Give for each continent the countries in which the life_expectancy is higher than the average life_expectancy of that continent.

```sql
SELECT continent, country, life_expectancy
FROM countries c
WHERE life_expectancy > (SELECT avg(life_expectancy) FROM countries WHERE continent = c.continent GROUP BY continent);
```

| continent | country    | life_expectancy |
| :-------- | :--------- | :-------------- |
| Africa    | Botswana   | 69,59           |
| Africa    | Congo      | 64,57           |
| Africa    | Cape Verde | 72,98           |
| Africa    | Djibouti   | 67,11           |
| Africa    | Algeria    | 76,88           |

## 8. Which country(ies) have the highest value for median_age?

```sql
SELECT country, median_age
FROM countries
WHERE median_age = (SELECT MAX(median_age) FROM countries);
```

| Japan |
| :---- |
| 48.2  |

## 9. State the countries where there are percentages of more people smoking than the average for the continent to which the country belongs.

- Assume that in each country the distribution is men/women = 50/50.

```sql
SELECT c.iso_code, c.country, c.continent,
(female_smokers + male_smokers) / 2 As percentage_smokers
FROM Countries c
WHERE (female_smokers + male_smokers) / 2 >
(SELECT avg((female_smokers + male_smokers) / 2)
FROM countries c1 WHERE c1.continent = c.continent);
```

| iso_code | country  | continent | percentage_smokers |
| :------- | :------- | :-------- | :----------------- |
| BWA      | Botswana | Africa    | 20,05              |
| COG      | Congo    | Africa    | 27                 |
| DZA      | Algeria  | Africa    | 15,55              |
| EGY      | Egypt    | Africa    | 25,15              |
| GMB      | Gambia   | Africa    | 15,95              |
| LSO      | Lesotho  | Africa    | 27,15              |

## 10. List all countries where the percentage of people have been vaccinated at least 1 time on 2021-12-01 is larger than average across that continent.

```sql
SELECT c.iso_code, c.country, c.continent, FORMAT(cd.people_vaccinated * 1.0 / c.population, 'P') As Percentage_Vaccinated
FROM Countries c JOIN CovidData cd ON c.iso_code = cd.iso_code
WHERE cd.report_date = '2021-12-01' AND (cd.people_vaccinated * 1.0 / c.population) > (SELECT (sum(cd1.people_vaccinated * 1.0)  / SUM(c1.population * 1.0)) FROM countries c1 JOIN covidData cd1 ON c1.iso_code = cd1.iso_code WHERE cd1.report_date = '2021-12-01' AND c1.continent = c.continent);
```

| iso_code | country           | continent | Percentage_Vaccinated |
| :------- | :---------------- | :-------- | :-------------------- |
| ETH      | Ethiopia          | Africa    | 6.877%                |
| GNQ      | Equatorial Guinea | Africa    | 17.100%               |
| MWI      | Malawi            | Africa    | 5.886%                |
| TUN      | Tunisia           | Africa    | 51.338%               |
