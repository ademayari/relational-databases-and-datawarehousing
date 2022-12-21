# Exercise 1

- Write a trigger in case there is an update on the hosp_patients column.
- In that case, the next weekly_hosp_admissions must be updated with the correct number.
- For example, if there is an update for Belgium for Tuesday 30/11/2021 from hosp_patients = 3750 to 3780, then the value of weekly_hosp_admissions for Tuesday 30/11/2021, Wednesday 01/12/2021, Thursday 02/12/2021, Friday 03/12/2021, Saturday 04/12/2021, Sunday 05/12/2021 and Monday 06/12/2021 must be updated (+ 30).

```sql
CREATE OR ALTER TRIGGER trigger_update_hosp_patients ON CovidData FOR update
AS
DECLARE @iso_code nvarchar(10), @report_date datetime, @hosp_patients_old bigint, @hosp_patients_new bigint, @difference int
SELECT @iso_code = iso_code, @report_date = report_date, @hosp_patients_new = hosp_patients
FROM inserted

SELECT @hosp_patients_old = hosp_patients
FROM deleted

SET @difference = @hosp_patients_new - @hosp_patients_old

UPDATE CovidData
SET weekly_hosp_admissions = weekly_hosp_admissions + @difference
WHERE iso_code = @iso_code AND report_date BETWEEN @report_date AND @report_date + 6
```

### Testcode

```sql
BEGIN TRANSACTION

SELECT \* FROM CovidData WHERE iso_code = 'BEL' AND report_date BETWEEN '2021-11-30' AND '2021-12-10'

UPDATE CovidData
SET hosp_patients = 3780
WHERE iso_code = 'BEL' AND report_date = '2021-11-30'

SELECT \* FROM CovidData WHERE iso_code = 'BEL' AND report_date BETWEEN '2021-11-30' AND '2021-12-10'

ROLLBACK

```

# Exercise 2

- Write a trigger in case there is an update on the hospital_beds_per_thousand column.
- (1) If the new number of hospital beds is more than doubling or halving the old number of hospital beds, (the old number of hospital beds is not NULL), then there is a rollback of the transaction.
- (2) If the new number of hospital beds is 50% larger or 50% smaller than the average number of hospital beds on that continent, then there is a rollback of the transaction.
- Write testcode
- (1) the number of hospital beds in Belgium is more than doubling or halving (e.g. new value is 14).
- (2) the new number of hospital beds in Belgium is 50% larger of 50% smaller than the average number of hospital beds on the continent (5.1) (e.g. new value is 9).
- (3) the new number of hospital beds in Belgium is correct (e.g. new value is 6.1).

```sql
CREATE OR ALTER TRIGGER update_hospital_beds ON Countries FOR update
AS
if update(hospital_beds_per_thousand)
BEGIN
	DECLARE @beds_new FLOAT = (SELECT hospital_beds_per_thousand From inserted)
	DECLARE @beds_old FLOAT = (SELECT hospital_beds_per_thousand From deleted)
	DECLARE @continent NVARCHAR(50) = (SELECT continent From inserted)

	IF @beds_old IS NOT NULL
	BEGIN
		IF @beds_new NOT BETWEEN @beds_old * 0.5 AND @beds_old * 2
		BEGIN
			ROLLBACK TRANSACTION
			RAISERROR ('The number of hospital beds is more than doubling or halving', 14,1)
		END
		DECLARE @avg_number_of_beds_continent FLOAT
		SET @avg_number_of_beds_continent = (SELECT AVG(hospital_beds_per_thousand) FROM countries WHERE continent = @continent)
		IF @beds_new NOT BETWEEN @avg_number_of_beds_continent * 0.5 AND  @avg_number_of_beds_continent * 1.5
		BEGIN
			ROLLBACK TRANSACTION
			RAISERROR ('The number of hospital beds is much larger or smaller than the average number of hospital beds on the continent', 14,1)
		END
	END
END


SELECT AVG(hospital_beds_per_thousand) FROM countries where continent = 'Europe'

```

### Testcode 1

```sql
BEGIN TRANSACTION
UPDATE countries SET hospital_beds_per_thousand = 14 WHERE country = 'Belgium'
SELECT * FROM countries WHERE country = 'Belgium'
ROLLBACK
```

### Testcode 2

```sql
BEGIN TRANSACTION
UPDATE countries SET hospital_beds_per_thousand = 9 WHERE country = 'Belgium'
SELECT * FROM countries WHERE country = 'Belgium'
ROLLBACK
```

### Testcode 3

```sql
BEGIN TRANSACTION
UPDATE countries SET hospital_beds_per_thousand = 6 WHERE country = 'Belgium'
SELECT * FROM countries WHERE country = 'Belgium'
ROLLBACK

```

# Exercise 3

- Write a trigger in case there is an update of the people_vaccinated.
- If the number of people_fully_vaccinated becomes larger than the people_vaccinated, there is a rollback of the transaction.
- In that case, the first report_date and iso_code for which the trigger goes wrong, are printed out.
- It's possible that with 1 UPDATE statement multiple rows are affected. Take this into account!

```sql
CREATE OR ALTER TRIGGER update_people_vaccinated
	ON CovidData
	FOR UPDATE
AS
BEGIN
	DECLARE @new_vaccinated BIGINT, @new_fully_vaccinated BIGINT, @iso_code NVARCHAR(10), @report_date datetime
	DECLARE c_tbl CURSOR
	FOR
	SELECT people_vaccinated, people_fully_vaccinated, iso_code, report_date
	FROM inserted

		OPEN c_tbl

		FETCH NEXT FROM c_tbl INTO @new_vaccinated, @new_fully_vaccinated, @iso_code, @report_date

		WHILE @@FETCH_STATUS = 0
		BEGIN
			IF @new_fully_vaccinated > @new_vaccinated
			BEGIN
				PRINT 'Error for ' + @iso_code + ' on ' + @report_date;
				ROLLBACK TRANSACTION;
			END
			FETCH NEXT FROM c_tbl INTO @new_vaccinated, @new_fully_vaccinated, @iso_code, @report_date
		END

		CLOSE c_tbl

		DEALLOCATE c_tbl
END
```

### Testcode

```sql
BEGIN TRANSACTION
UPDATE CovidData SET people_vaccinated = 4000000 WHERE iso_code = 'BEL' AND report_date >= '2021-05-01 00:00:00.0000000'
ROLLBACK
```
