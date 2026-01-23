# Exercise 02: World Database – Joins, Grouping, and Data Quality

- Name: Branton Dawson
- Course: Database for Analytics
- Module: 2
- Database Used: World Database (PostgreSQL)

---

## Instructions

- Answer each question below using SQL executed against the **World database**.
- All SQL commands **must be run by you**.
- For each SQL-based question:
  - Include the SQL command in a fenced code block
  - Include a **screenshot** showing the command and its results
- Store screenshots in the `screenshots/` folder and embed them below each answer.

---

## Question 1

When importing records from `worldPGSQL.sql`, **how many cities were imported**?

### Answer
_4079_

### Screenshot
_Show evidence of how you determined this (for example, a COUNT query)._

```sql
-- SELECT COUNT(name) FROM city
```

![Q1 Screenshot](screenshots/q1_city_count.png)

---

## Question 2

Using the World database, write the SQL command to **display each country name along with the name of each language spoken in that country**.

### SQL

```sql
-- SELECT country.name AS "Country", language
-- FROM countrylanguage
-- INNER JOIN country ON country.code=countrylanguage.countrycode;
```

### Screenshot

![Q2 Screenshot](screenshots/q2_country_languages.png)

---

## Question 3

Using the World database, write the SQL command to **display each country name along with the name of each official language spoken in that country**.

### SQL

```sql
-- SELECT country.name AS "Country", language
-- FROM countrylanguage
-- INNER JOIN country ON country.code=countrylanguage.countrycode AND isofficial='T';
```

### Screenshot

![Q3 Screenshot](screenshots/q3_official_languages.png)

---

## Question 4

Consider the following two SQL statements:

```sql
SELECT *
FROM country, countrylanguage
WHERE country.code = countrylanguage.countrycode;
```

```sql
SELECT *
FROM country
LEFT OUTER JOIN countrylanguage
ON country.code = countrylanguage.countrycode;
```

**In your own words**, describe what data the **second query returns that the first query does not**.

### Answer
_The second query includes every row in the country table where country.code = countrylanguage.countrycode and NULL values for the values that are not found, where the first query will return all rows where country.code = countrylanguage.countrycode match._

---

## Question 5

Using the World database, write the SQL command to **list all different forms of government** found in the data.
Do **not** repeat any form of government more than once.

### SQL

```sql
-- select distinct(governmentform) from country
-- order by governmentform;
```

### Screenshot

![Q5 Screenshot](screenshots/q5_government_forms_ex02.png)

---

## Question 6

Using the World database, write the SQL command to **list all names of cities and countries in one column**.
Label the column **"City or Country Name"**.

### SQL

```sql
-- (select name as "City or Country Name" from city) union
-- (select name as "City or Country Name" from country)
-- order by "City or Country Name"
```

### Screenshot

![Q6 Screenshot](screenshots/q6_union_city_country.png)

---

## Question 7

Using the World database, write the SQL command to **list all countries by name**, along with the **number of languages spoken in each country**.
Be sure to **sort by country name**.

### SQL

```sql
-- SELECT country.name AS "Country", count(countrylanguage.language) AS "Number of Languages"
-- FROM country
-- JOIN countrylanguage
-- ON country.code = countrylanguage.countrycode
-- GROUP BY country.name
-- ORDER BY country.name;
```

### Screenshot

![Q7 Screenshot](screenshots/q7_language_count_by_country.png)

---

## Question 8

Using the World database, write the SQL command to **list all languages**, along with the **number of countries where each language is spoken**.
Be sure to **sort by language name**.

### SQL

```sql
-- SELECT countrylanguage.language AS "Languages", count(country.name) AS "Number of Countries Spoken"
-- FROM countrylanguage
-- JOIN country
-- ON countrylanguage.countrycode = country.code
-- GROUP BY countrylanguage.language
-- ORDER BY countrylanguage.language
```

### Screenshot

![Q8 Screenshot](screenshots/q8_language_country_count.png)

---

## Question 9

Using the World database, write the SQL command to **list countries that have more than two official languages**, along with the **number of official languages spoken**.

*Hint: There are 8 such countries in this dataset.*

### SQL

```sql
-- SELECT country.name AS "Country", count(countrylanguage.language) AS "Number of Languages"
-- FROM country
-- JOIN countrylanguage
-- ON country.code = countrylanguage.countrycode
-- WHERE countrylanguage.isofficial = 'T'
-- GROUP BY country.name
-- HAVING COUNT(countrylanguage.isofficial) > 2
-- ORDER BY country.name;
```

### Screenshot

![Q9 Screenshot](screenshots/q9_multiple_official_languages.png)

---

## Question 10

Using the World database, write the SQL command to **find cities where the district value is missing**.

*Hint: Use `LIKE` and the dash (`-`) since some rows use that instead of actual data.*

### SQL

```sql
-- select name, district from city
-- where
--   district like ' %'
--   or district IS NULL
--   or btrim(district) = ''
--   or district = '-'
--   or length(btrim(district)) = 1
-- order by name Asc;
```

### Screenshot

![Q10 Screenshot](screenshots/q10_missing_districts.png)

---

## Question 11

Using the World database, write the SQL command to **calculate the percentage of cities with missing district values**.

*Hint: The result should be approximately 0.4%.*

### SQL

```sql
-- WITH missing_city AS (
-- SELECT NAME, district FROM city
-- where
--   district LIKE ' %'
--   OR district IS NULL
--   OR btrim(district) = ''
--   OR district = '-'
--   OR LENGTH(btrim(district)) = 1
-- )
-- SELECT
--     COUNT(*) AS missing_city, (SELECT COUNT(*) FROM city) as Total_Cities,
--     (COUNT(*)::numeric / (SELECT COUNT(*) FROM city)) * 100
--         AS percentage_missing
-- FROM missing_city;
```

### Screenshot

![Q11 Screenshot](screenshots/q11_missing_district_percentage.png)
