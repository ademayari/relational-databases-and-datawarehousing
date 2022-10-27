# 1. We want to investigate the height of the waves in some countries. You can only compare countries if you take into account there population.

## Part 1

- Show for Belgium, France and the Netherlands a ranking (per country) of the days with the most new cases per 100.000 inhabitants.
- Show only the top 5 days per country.

| report_date             | country     | cases_per_100000  | rank_new_cases |
| :---------------------- | :---------- | :---------------- | :------------- |
| 2022-01-24 00:00:00.000 | Belgium     | 1147.491122589843 | 1              |
| 2022-01-31 00:00:00.000 | Belgium     | 853.775347234699  | 2              |
| 2022-01-17 00:00:00.000 | Belgium     | 630.105703636088  | 3              |
| 2022-01-27 00:00:00.000 | Belgium     | 587.027504540361  | 4              |
| 2022-01-20 00:00:00.000 | Belgium     | 579.832044024870  | 5              |
| 2022-01-25 00:00:00.000 | France      | 743.746866963714  | 1              |
| 2022-01-18 00:00:00.000 | France      | 687.891885310768  | 2              |
| 2022-01-19 00:00:00.000 | France      | 645.558847385135  | 3              |
| 2022-01-26 00:00:00.000 | France      | 632.192311286037  | 4              |
| 2022-01-20 00:00:00.000 | France      | 631.906657264653  | 5              |
| 2022-02-07 00:00:00.000 | Netherlands | 2215.087159017472 | 1              |
| 2022-01-30 00:00:00.000 | Netherlands | 653.504837276264  | 2              |
| 2022-01-31 00:00:00.000 | Netherlands | 616.184829594480  | 3              |
| 2022-02-05 00:00:00.000 | Netherlands | 552.573694641163  | 4              |
| 2022-02-09 00:00:00.000 | Netherlands | 505.377772927813  | 5              |

## Part 2

- Give the top 10 of countries with more than 1.000.000 inhabitants with the highest number new cases per 100.000 inhabitants.

| country     | max_cases_per_100000 | rank_max_cases_per_100000 |
| :---------- | :------------------- | :------------------------ |
| Greece      | 3544.865186663988    | 1                         |
| Israel      | 2618.609406952965    | 2                         |
| Mauritius   | 2356.316964916744    | 3                         |
| Netherlands | 2215.087159017472    | 4                         |
| Botswana    | 1734.327810315195    | 5                         |
| Sweden      | 1367.941190684122    | 6                         |
| South Korea | 1211.021872565548    | 7                         |
| Belgium     | 1147.491122589843    | 8                         |
| Slovenia    | 1122.419870276126    | 9                         |
| Costa Rica  | 1078.136380379809    | 10                        |

# 2. Make a ranking (high to low) of countries for the total number of deaths until now relative to the number of inhabitants. Show the rank number (1,2,3, ...), the country, relative number of deaths

| country                | relative_number_of_deaths | rank_deaths |
| :--------------------- | :------------------------ | :---------- |
| Peru                   | 0.006503                  | 1           |
| Bulgaria               | 0.005461                  | 2           |
| Bosnia and Herzegovina | 0.004941                  | 3           |
| Hungary                | 0.004920                  | 4           |
| North Macedonia        | 0.004571                  | 5           |
| Montenegro             | 0.004424                  | 6           |
| Georgia                | 0.004246                  | 7           |
| Croatia                | 0.004122                  | 8           |
| Czechia                | 0.003820                  | 9           |
| Slovakia               | 0.003740                  | 10          |
| Romania                | 0.003496                  | 11          |

# 3. In the press conferences, Sciensano always gives updates on the weekly average instead of the absolute numbers, to eliminate weekend, ... effects.

## 1. Calculate for each day the weekly average of the number of new_cases and new_deaths in Belgium

## 2. Calculate for each day the relative difference with the previous day in Belgium for the weekly average number of new cases

## 3. Give the day with the highest relative difference of weekly average number of new cases in Belgium after 2020-04-01

| report_date             | new_cases | new_deaths | weekly_avg_new_cases | weekly_avg_new_deaths | weekly_avg_new_cases_previous | relative_difference |
| :---------------------- | :-------- | :--------- | :------------------- | :-------------------- | :---------------------------- | :------------------ |
| 2022-09-13 00:00:00.000 | 6907      | 30         | 1619.000000          | 6.285714              | 632.285714                    | 0.609459            |
| 2022-06-02 00:00:00.000 | 6196      | 14         | 1598.000000          | 5.857142              | 712.857142                    | 0.553906            |
| 2022-08-05 00:00:00.000 | 12098     | 33         | 3292.000000          | 9.714285              | 1563.714285                   | 0.524995            |
| 2022-09-15 00:00:00.000 | 7484      | 20         | 2055.857142          | 7.142857              | 986.714285                    | 0.520047            |
| 2022-06-07 00:00:00.000 | 5944      | 31         | 1734.285714          | 6.428571              | 885.142857                    | 0.489621            |
| 2022-07-28 00:00:00.000 | 16094     | 49         | 4703.428571          | 15.714285             | 2404.285714                   | 0.488822            |

# 4 The main reason for the lockdowns was to prevent the hospital system from collapsing (i.e. too much patients on IC) Give those weeks for which the number of hospitalized patients in Belgium doubled compared to the week before.

## Step 1: Add 2 extra columns with the weeknumber and year of each date. Use DATEPART(WEEK,report_date) for the weeknumber.

## Step 2: Calculate the average number of hosp_patients during that week.

## Step 3: Calculate the relative difference between each 2 weeks.

## Step 4: Give those weeks for which the number of hosp_patients rose with 50%.

| report_week | report_year | avg_number_hosp_patients | avg_number_hosp_patients_previous_week | relative_change     |
| :---------- | :---------- | :----------------------- | :------------------------------------- | :------------------ |
| 13          | 2020        | 2759                     | 729                                    | 2.78463648834019204 |
| 14          | 2020        | 5161                     | 2759                                   | 0.87060529177238129 |
| 39          | 2020        | 546                      | 351                                    | 0.55555555555555555 |
| 42          | 2020        | 1789                     | 1052                                   | 0.70057034220532319 |
| 43          | 2020        | 3376                     | 1789                                   | 0.88708775852431525 |
| 44          | 2020        | 5813                     | 3376                                   | 0.72186018957345971 |

# 5 Rank the countries per continent based on the percentage of the population that is fully vaccinated.

- The ranking shows the countries with the highest percentage of fully vaccinated people at the top and the countries with the lowest percentage at the bottom.

| iso_code | country      | continent | percentage_fully_vaccinated | ranking |
| :------- | :----------- | :-------- | :-------------------------- | :------ |
| SYC      | Seychelles   | Africa    | 82.10%                      | 1       |
| RWA      | Rwanda       | Africa    | 79.00%                      | 2       |
| MUS      | Mauritius    | Africa    | 76.78%                      | 3       |
| MAR      | Morocco      | Africa    | 62.90%                      | 4       |
| SHN      | Saint Helena | Africa    | 57.93%                      | 5       |
| BWA      | Botswana     | Africa    | 57.30%                      | 6       |

# 6 The Pareto principle, known as the "80-20" rule, says that 20% of the population owns 80% of the wealth. We are going to investigate whether 80% of the wealth is in 20% of the countries worldwide. Make the overview below to easily check this.

- wealth_land = population \* gdp_per_capita / 1000000 ( /1000000 serves to not make the numbers too large, this does not change the overview).
- Only those countries whose population is not NULL and the gdp_per_capita is not NULL are included in the overview.
- Note that the countries are sorted in descending order by wealth_land.

| country        | rijkdom_land     | nummer_land | totaal_aantal_landen | cumulatieve_som_rijkdom | totale_rijkdom_wereldwijd | percentage_landen | percentage_rijkdom |
| :------------- | :--------------- | :---------- | :------------------- | :---------------------- | :------------------------ | :---------------- | :----------------- |
| China          | 22109088,3712806 | 1           | 193                  | 22109088,3712806        | 118414404,940934          | 0.52%             | 18.67%             |
| United States  | 18052468,367773  | 2           | 193                  | 40161556,7390536        | 118414404,940934          | 1.04%             | 33.92%             |
| India          | 8954985,60374624 | 3           | 193                  | 49116542,3427999        | 118414404,940934          | 1.55%             | 41.48%             |
| Japan          | 4916261,25491951 | 4           | 193                  | 54032803,5977194        | 118414404,940934          | 2.07%             | 45.63%             |
| Germany        | 3794754,95847439 | 5           | 193                  | 57827558,5561938        | 118414404,940934          | 2.59%             | 48.83%             |
| Russia         | 3613650,42489899 | 6           | 193                  | 61441208,9810928        | 118414404,940934          | 3.11%             | 51.89%             |
| Indonesia      | 3092141,29731427 | 7           | 193                  | 64533350,278407         | 118414404,940934          | 3.63%             | 54.50%             |
| Brazil         | 3018046,22345833 | 8           | 193                  | 67551396,5018654        | 118414404,940934          | 4.15%             | 57.05%             |
| United Kingdom | 2711454,04537782 | 9           | 193                  | 70262850,5472432        | 118414404,940934          | 4.66%             | 59.34%             |
| France         | 2608363,24546742 | 10          | 193                  | 72871213,7927106        | 118414404,940934          | 5.18%             | 61.54%             |
| Mexico         | 2258286,93890118 | 11          | 193                  | 75129500,7316118        | 118414404,940934          | 5.70%             | 63.45%             |
| Turkey         | 2137067,91251698 | 12          | 193                  | 77266568,6441288        | 118414404,940934          | 6.22%             | 65.25%             |

# 7 Rank all countries showing from what date in a country 0.1% of the population has already received a booster shot.

- A ranking is made per continent and for the whole world
- Sort ascending by date
- Where is Belgium in Europe and in the world?

| iso_code | country              | continent     | report_date             | ranking_continent | ranking_world |
| :------- | :------------------- | :------------ | :---------------------- | :---------------- | :------------ |
| EST      | Estonia              | Europe        | 2021-02-26 00:00:00.000 | 1                 | 1             |
| FRA      | France               | Europe        | 2021-06-15 00:00:00.000 | 2                 | 2             |
| TUR      | Turkey               | Asia          | 2021-06-30 00:00:00.000 | 1                 | 3             |
| DOM      | Dominican Republic   | North America | 2021-07-05 00:00:00.000 | 1                 | 4             |
| ARE      | United Arab Emirates | Asia          | 2021-07-15 00:00:00.000 | 2                 | 5             |
| ISR      | Israel               | Asia          | 2021-07-31 00:00:00.000 | 3                 | 6             |
| THA      | Thailand             | Asia          | 2021-08-05 00:00:00.000 | 4                 | 7             |
| KHM      | Cambodia             | Asia          | 2021-08-09 00:00:00.000 | 5                 | 8             |
| URY      | Uruguay              | South America | 2021-08-09 00:00:00.000 | 1                 | 8             |
| CHL      | Chile                | South America | 2021-08-11 00:00:00.000 | 2                 | 10            |
