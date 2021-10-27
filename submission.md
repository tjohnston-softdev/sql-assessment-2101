# SQL Assessment

---

### Question 1

Dr. Brown wants to see all patients with Asthma (contained in `asthma_pts` table). She would also like to see if the patients have COPD or not (contained in `copd_pts` table). She Runs the query below:

```sql
SELECT a.patient_id, a.asthma_dx_dte, b.copd_dx_dte
FROM asthma_pts AS a
	INNER JOIN copd_pts AS b
		ON a.patient_id = b.patient_id;
```

Only 100 rows are returned, and She expected 3-4 thousand records. Can you identify what went wrong with the query?
\
**Answer:**

This query appears to retrieve the patients that are diagnosed with both Asthma and COPD. The  query itself is not wrong for what it is, but the question states that Dr. Brown wants to see patients  who do have Asthma, and whether those particular patients have COPD or not. This implies that  COPD is optional rather than required. What Dr. Brown should had done is use a `LEFT JOIN` instead of an `INNER JOIN`. The query will retrieve all patients with Asthma while also indicating whether COPD has been diagnosed. If a patient has Asthma but not COPD, the `copd_dx_date` cell will be null.

---

### Question 02

In order to improve her database performance, Dr. Brown tries to add a primary key to the table below, called `clinical_events`, on the `patient_id` field. The database returns an error, can you explain why?

| patient_id | event_date | read_code |
|---|---|---|
| 9876 | 2006-10-11 | H33.. |
| 9876 | 2003-10-11 | H441. |
| 9876 | 2008-05-19 | H33.. |
| 4666 | 2001-01-01 | H33.. |
\
**Answer:**

The reason that an error was flagged when Dr. Brown attempted to add the Primary Key is because the values in a PK column must be unique. If this example data is any indication, three patients have the ID number 9876. Each row in the `clinical_events` table must have a different `patient_id` column value. In order for this data to be valid, the IDs would have to be: 9876, 9877, 9878, and 4666. We can assume that the last one is unique.

---

### Question 03

After exploring the data, Dr. Brown wants to compare the average age at diagnosis for the asthma patients divided by gender.

Based on an excerpt from the `asthma_patients` table. How could she query this information?

| patient_id | birth_year | gender | diag_date |
|---|---|---|---|
| 9876 | 1990 | F | 1999-05-06 |
| 4666 | 1975 | M | 1994-12-08 |
\
**Answer:**

This is easily done but I should point out that while the full date is stored for diagnosis, this table only stores the patient's birth_year and not the exact Date Of Birth. In order to get a truly accurate age at diagnosis, the full DOB should be stored and not just the year. Therefore, this query will assume the patient's age as of that year (eg. Patient 9876 turns 9 in 1999. We just don't know the exact date)

In the context of this question, the 'age' is simply the `birth_year` subtracted from `YEAR(diag_date)`. Because average calculations may return decimals, I decided to round to the nearest whole.

```sql
SELECT
	gender,
	ROUND(AVG(YEAR(diag_date) - birth_year)) AS avg_age
FROM asthma_patients
GROUP BY gender;
```

---

### Question 04

Dr Brown wants to return the 10 patient identifiers (`patient_id`) with the most recent `event_date` fields from the `clinical_events` table in [question 02](#question02). Write a query to retrieve that data.
\
**Answer:**

I have also included the corresponding `event_date` for readability.

```sql
SELECT patient_id, event_date
FROM clinical_events
ORDER BY event_date DESC
LIMIT 10;
```

---

### Question 05

Dr Brown wants to count the number of patients in the `clinical_events` table from [question 02](#question02). She runs this query:

```sql
SELECT COUNT(*) FROM clinical_events;
```

What is wrong with this query in terms of what Dr. Brown is trying to achieve? What should it be instead?
\
**Answer:**

When you use `COUNT(*)`, it counts the number of rows that match the query, not necessarily the number of patients. This is okay if the `patient_id` has a unique constraint but if it doesn't, the same patient can potentially have multiple rows. Hence, they are counted multiple times.

The query should be:

```sql
SELECT COUNT(DISTINCT patient_id) AS patientCount
FROM clinical_events
```

That way, the query will count the number of different patient IDs that appear. Each ID will only be counted once regardless if it is unique or not.

---

### Question 06

Dr. Brown looks at the mean height recorded in the database by summing all recorded heights and dividing by the number of recordings. She is surprised to see that the mean height is 2.01 and the maximum is 245. What has she not taken into account about how the data is being recorded?
\
**Answer:**

I don't have any example data so I can't cite anything specific. Based on the question, it looks like that there is inconsistency in how the different patient heights were recorded. Some of them could have been entered in meters (2.01m), and others could have been entered in centimetres (245cm).

What matters most here is consistency. All patients should have their height recorded in the same way. Otherwise, derived data such as average height may be inaccurate. What Dr. Brown could have done to mitigate this is to convert or re-enter all of the heights to her preferred unit and average them there.

---

### Question 07

Dr. Brown frequently runs the query below:

```sql
SELECT read_code
FROM clinical_events AS a
INNER JOIN therapy_events AS b
	ON a.patient_id = b.patient_id
	AND a.practice_id = b.practice_id
WHERE a.event_date <= ‘2018-10-01’;
```

The query takes a long time to run and someone suggests adding indexes to the tables. What fields would be best to index on? What type of index would you use?
\
**Answer:**

The question as to which columns to index can be quite subjective. It can depend on factors such as 'Is the column a Primary Key?', 'How often are we going to look up this column', and 'What is the trade-off between storage used and performance gained?'.

For starters, columns that are Primary Keys are indexed by default, so we don't have to worry about `patient_id` or `practice_id`. What I am concerned about is the `event_date` column. This query is used to retrieve events that happened before or on a certain date. I think creating a clustered index on the `event_date` column. This is because within that separate index, the event dates will be sorted in chronological order regardless of how the event rows themselves are sorted. Therefore, when the query performs the `WHERE` condition, it can consult this index and isolate all dates that meet the target.

In summary, I would place a clustered index on the `event_date` column so that the dates are sorted in chronological order in a separate file where it would be less taxing for the DMBS to isolate the target records.

---

### Question 08

Dr. Brown wants to count all the patients with either COPD or Asthma. She runs this query:

```sql
SELECT patient_id
FROM asthma_pts
UNION ALL
SELECT patient_id
FROM copd_pts;
```

The query returns 4045 rows and Dr. Brown assumes that this is the number of patients with either COPD or Asthma in her database, is she correct?
\
**Answer:**

Yes, this query is correct. `UNION ALL` combines the result of two queries while removing duplicate values. The results will contain patients that appear in either or both of the two tables. In other words, a patient could have Asthma, COPD, or both.

If she just wants to see a count and not a full list of IDs, she should change `SELECT` to:

```sql
SELECT COUNT(DISTINCT patient_id)
--etc
```

---

### Question 09

Dr. Brown wants to find all female patients with records that have the read code "H441." in the 10 years since their asthma diagnosis. Using the `asthma_patients`  table from [question 03](#question03) and the `clinical_events`  table from [question 02](#question02), construct a query to produce the information she requires.
\
**Answer:**

Assuming the `patient_id` column is the Primary Key of both tables:

```sql
SELECT
	asm.patient_id,
	asm.diag_date,
	evt.event_date,
	TIMESTAMPDIFF(year, asm.diag_date, evt.event_date) AS yearsSinceDiag
FROM clinical_events evt, asthma_patients asm
WHERE
	(evt.patient_id = asm.patient_id) AND
	(asm.gender = "F") AND
	(evt.read_code = "H441.")
GROUP BY evt.patient_id, asm.diag_date, evt.event_date
HAVING (yearsSinceDiag >= 0 AND yearsSinceDiag < 10)
ORDER BY evt.patient_id;
```

---

### Question 10

New data is to be loaded into the database from a comma-separated values (CSV) file. However, when the import script is run, an error is encountered. Examine the script and the csv file below to troubleshoot the likely source of the problem.

**import_file.csv**

```
patient_id,Event_date,Read_code,Read_term
9876,”2016-10-30”,”H33..”,” Asthma - new diagnosis”
1892,”2003-10-11”,”H441.”,”No “Cannabinosis”, other”
7482,”2013-04-21”,”XE2Ne”, “Diabetes monitored”
```
\
**import_script.sql**

```sql
BULK INSERT #temp_import FROM 'PATH'
WITH
(
	FIELDTERMINATOR = ',',
	FIRSTROW = 2,
	ROWTERMINATOR = '\n',
	FIRSTROW = 2,
	ROWTERMINATOR = '\n'
);
```
\
**Answer:**

First, the `BULK INSERT` line should specify the table that the CSV data is being imported into. Instead of '#temp-import', you should write `database_name.schema_name.table_name`. It is also worth specifying the `FORMAT = 'CSV'` in the `WITH` object. You might also want to change the `ROWTERMINATOR` option to `\r\n`.

Regarding the CSV data, there are three rows with four fields. Each field in a row is comma-separated. Quotation marks are used to escape any commas inside the text. First, remove quotation marks from the `event_date` field values. Although they are strings being converted into dates, the quotation marks are only necessary when the string actually contains commas.

The same applies for `read_code` and `read_term`. However, row 2 has a `read_term` value with both quotation marks and commas. First, the word [Cannabinosis] should be enclosed with inverted commas rather than quotation marks. This prevents the CSV parser from getting confused between the end of the string, and literal characters.

When it comes to debugging these sorts of scripts, trial and error may be the best approach. No two import scenarios are the same and there seems to be no universal way to parse a CSV file. The best thing you can do is tailor your script to accommodate any quirks that your target CSV file may have.

Here is the corrected CSV data:

```
patient_id,event_date,read_code,read_term
9876,2016-10-30,H33..,Asthma - new diagnosis
1892,2003-10-11,H441.,"No 'Cannabinosis', other"
7482,2013-04-21,XE2Ne,Diabetes monitored
```

\
**Append (2021-10-27):**

If a property in the `WITH` object is specified multiple times, it may cause an error, so please get rid of duplicate properties if there are any.

The person who put together the assessment document inserted the sample documents side-by-side in an image. When I attempted to view the document in [LibreOffice](https://www.libreoffice.org/), the SQL script in the image appeared all glitchy and I honestly couldn't tell what I was supposed to be reading. That is why I did not notice the duplicate properties when I wrote the original submission. I only realized it just now as I was converting it to markdown. I wrote several paragraphs detailing every possibility *except* for the one they actually wanted me to look at. I ran the original `.docx` file through a [PDF converter](https://smallpdf.com/word-to-pdf) and the image came out correctly. I was able to understand the script a little better.

I don't consider my mishap to be *their* fault. It is indeed *my* fault for using crappy knock-off software as opposed to the genuine Microsoft product which *every*body uses. I love it how everybody just low-key assumes that we are using the Microsoft ecosystem and have ready-access to the full office suite at any time from any device. In other words, they just send a `.docx` file and subconsciously assume that we know what to do with it. In all fairness, I have been a Microsoft user my whole life and I do use the free [cloud version](https://www.microsoft.com/en-au/microsoft-365/onedrive/online-cloud-storage) of Office when I cannot avoid it but.....

[I decided to cut off my rant here.]

tl;dw: I always use free and/or open source software as long as it is possible, and I suggest that you do the same.
