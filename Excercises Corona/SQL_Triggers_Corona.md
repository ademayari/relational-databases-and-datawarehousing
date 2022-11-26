-- Exercise 1
-- Write a trigger in case there is an update on the hosp_patients column.
-- In that case, the next weekly_hosp_admissions must be updated with the correct number
-- For example, if there is an update for Belgium for Tuesday 30/11/2021 from
-- hosp_patients = 3750 to 3780, then the value of weekly_hosp_admissions for Tuesday 30/11/2021,
-- Wednesday 01/12/2021, Thursday 02/12/2021, Friday 03/12/2021, Saturday 04/12/2021, Sunday 05/12/2021 and Monday 06/12/2021 must be updated (+ 30).

-- Testcode
BEGIN TRANSACTION

SELECT \* FROM CovidData WHERE iso_code = 'BEL' AND report_date BETWEEN '2021-11-30' AND '2021-12-10'

UPDATE CovidData
SET hosp_patients = 3780
WHERE iso_code = 'BEL' AND report_date = '2021-11-30'

SELECT \* FROM CovidData WHERE iso_code = 'BEL' AND report_date BETWEEN '2021-11-30' AND '2021-12-10'

ROLLBACK

-- Exercise 2
-- Write a trigger in case there is an update on the hospital_beds_per_thousand column.
-- (1) If the new number of hospital beds is more than doubling or halving the old number of hospital beds
-- (the old number of hospital beds is not NULL), then there is a rollback of the transaction
-- (2) If the new number of hospital beds is 50% larger or 50% smaller than the average number of hospital beds on that continent,
-- then there is a rollback of the transaction
-- Write testcode
-- (1) the number of hospital beds in Belgium is more than doubling or halving (e.g. new value is 14)
-- (2) the new number of hospital beds in Belgium is 50% larger of 50% smaller than the average number of hospital beds on the continent (5.1) (e.g. new value is 9)
-- (3) the new number of hospital beds in Belgium is correct (e.g. new value is 6.1)

GO;

-- Exercise 3
-- Write a trigger in case there is an update of the people_vaccinated.
-- If the number of people_fully_vaccinated becomes larger than the people_vaccinated, there is a rollback of the transaction
-- In that case, the first report_date and iso_code for which the trigger goes wrong, are printed out
-- It's possible that with 1 UPDATE statement multiple rows are affected. Take this into account!

GO;

-- Testcode
BEGIN TRANSACTION
UPDATE CovidData SET people_vaccinated = 4000000 WHERE iso_code = 'BEL' AND report_date >= '2021-05-01 00:00:00.0000000'
ROLLBACK
