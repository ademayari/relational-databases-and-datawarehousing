## 1. Give the names of all countries for which a larger percentage of people was vaccinated (not fully vaccinated) than for Belgium on 1 april 2021

```sql
WITH cte_1 (percentage_vaccinated_Belgium)
AS
(SELECT cd.people_vaccinated * 1.0 / c.population
FROM CovidData cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE report_date = '2021-04-01' and c.country = 'Belgium')

SELECT c.country, cd.people_vaccinated * 1.0 / c.population AS percentage_people_vaccinated
FROM CovidData cd JOIN Countries c ON cd.iso_code = c.iso_code JOIN cte_1 ON cd.people_vaccinated * 1.0 / c.population > percentage_vaccinated_Belgium
WHERE report_date = '2021-04-01';
```

| country  | percentage_people_vaccinated |
| :------- | :--------------------------- |
| Austria  | 0.142731363855               |
| Bahrain  | 0.296061591436               |
| Barbados | 0.221526686779               |
| Bhutan   | 0.557554814719               |
| Canada   | 0.138320243613               |
| Chile    | 0.361848064282               |

## 2. Give for each month the percentage of fully vaccinated people in Belgium at the end of the month

**SOLUTION WITH CTE (COMMON TABLE EXPRESSION)**

```sql
WITH cte_end_month([month], [year], endday)
AS
(SELECT month(report_date), year(report_date), MAX(report_date)
FROM CovidData cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE c.country = 'Belgium'
GROUP BY month(report_date), year(report_date))

SELECT [month], [year], FORMAT(cd.people_fully_vaccinated * 1.0 / c.population, 'P')
FROM CovidData cd JOIN Countries c ON cd.iso_code = c.iso_code
JOIN cte_end_month cte ON cte.endday = cd.report_date
WHERE c.country = 'Belgium' AND cd.people_fully_vaccinated IS NOT NULL;
```

**SOLUTION WITHOUT CTE**

```sql
SELECT MONTH(report_date), YEAR(report_date), FORMAT(cd.people_fully_vaccinated * 1.0 / c.population, 'P')
FROM CovidData cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE c.country = 'Belgium' AND cd.people_fully_vaccinated IS NOT NULL AND report_date = EOMONTH(report_date);
```

| month | year | (No column name) |
| :---- | :--- | :--------------- |
| 12    | 2020 | 0.00%            |
| 1     | 2021 | 0.24%            |
| 2     | 2021 | 2.95%            |
| 3     | 2021 | 4.90%            |
| 4     | 2021 | 7.56%            |
| 5     | 2021 | 19.24%           |
| 6     | 2021 | 35.23%           |
| 7     | 2021 | 59.64%           |
| 8     | 2021 | 70.52%           |
| 9     | 2021 | 72.86%           |
| 10    | 2021 | 74.15%           |
| 11    | 2021 | 75.11%           |
| 12    | 2021 | 75.90%           |
| 1     | 2022 | 76.42%           |
| 2     | 2022 | 78.15%           |
| 3     | 2022 | 78.57%           |
| 4     | 2022 | 78.62%           |
| 5     | 2022 | 78.66%           |
| 6     | 2022 | 78.69%           |
| 7     | 2022 | 78.71%           |
| 8     | 2022 | 78.73%           |

## 3. What is the percentage of the total amount of new_cases that died in the following periods in Belgium; march 2020 - may 2020 / june 2020 - august 2020 / september 2020 - november 2020 / december 2020 - february 2021 / march 2021 - may 2021 / june 2021 - august 2021

```sql
WITH cte_1 (report_date, new_cases, new_deaths, periode)
AS
(SELECT report_date, new_cases, new_deaths,
CASE
WHEN report_date BETWEEN '2020-03-01' AND '2020-05-31' THEN 'march 2020 - may 2020'
WHEN report_date BETWEEN '2020-06-01' AND '2020-08-31' THEN 'june 2020 - august 2020'
WHEN report_date BETWEEN '2020-09-01' AND '2020-11-30' THEN 'september 2020 - november 2020'
WHEN report_date BETWEEN '2020-12-01' AND '2021-02-28' THEN 'december 2020 - february 2021'
WHEN report_date BETWEEN '2021-03-01' AND '2021-05-31' THEN 'march 2021 - may 2021'
WHEN report_date BETWEEN '2021-06-01' AND '2021-08-31' THEN 'june 2021 - august 2021'
END AS periode
FROM CovidData cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE country = 'Belgium')

SELECT periode, FORMAT(SUM(new_deaths * 1.0) / SUM(new_cases * 1.0), 'P') AS sterfpercentage
FROM cte_1
WHERE periode IS NOT NULL
GROUP BY periode
ORDER BY SUM(new_deaths * 1.0) / SUM(new_cases * 1.0) DESC;
```

| march 2020 - may 2020 16.22%         |
| :----------------------------------- |
| june 2020 - august 2020 1.59%        |
| september 2020 - november 2020 1.37% |
| december 2020 - february 2021 2.80%  |
| march 2021 - may 2021 0.99%          |
| june 2021 - august 2021 0.35%        |

## 4. Which country(ies) was(were) the first to have 50% of the population fully vaccinated

```sql
WITH cte1(min_report_date)
AS
(SELECT MIN(report_date)
FROM CovidData cd JOIN Countries c ON cd.iso_code = c.iso_code
WHERE cd.people_fully_vaccinated * 1.0 / c.population > 0.5)

SELECT c.country
FROM CovidData cd JOIN Countries c ON cd.iso_code = c.iso_code JOIN cte1 cte ON report_date = min_report_date
WHERE cd.people_fully_vaccinated * 1.0 / c.population > 0.5;
```

- Gibraltar

## 5. Give the countries in which a total of more new_cases were identified in the first half of 2021 than in the second half of 2021

```sql
WITH cte_1 AS
(SELECT iso_code, SUM(new_cases) As new_cases_jan_jun_2021 FROM CovidData WHERE report_date BETWEEN '2021/01/01' AND '2021/06/30' GROUP BY iso_code),

cte_2 AS
(SELECT iso_code, SUM(new_cases) As new_cases_jul_dec_2021 FROM CovidData WHERE report_date BETWEEN '2021/07/01' AND '2021/12/31' GROUP BY iso_code)

SELECT cte_1.iso_code, c.continent, c.country FROM cte_1 JOIN cte_2 ON cte_1.iso_code = cte_2.iso_code AND cte_1.new_cases_jan_jun_2021 > cte_2.new_cases_jul_dec_2021 JOIN countries c
ON c.iso_code = cte_1.iso_code
ORDER BY iso_code;
```

| iso_code | continent | country                |
| :------- | :-------- | :--------------------- |
| AFG      | Asia      | Afghanistan            |
| ARE      | Asia      | United Arab Emirates   |
| ARG      | South     | America Argentina      |
| BFA      | Africa    | Burkina Faso           |
| BHR      | Asia      | Bahrain                |
| BIH      | Europe    | Bosnia and Herzegovina |
