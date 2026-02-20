# Module 7 - Final Exercise

Final Exercise / Final Project - Show Us Your Data

- Name: Branton Dawson
- Course: Database for Analytics
- Module: 7

---

## Overview

In this exercise, you will show how you created your database and the steps you went through to get there.

During Module 7, no late work is accepted, regardless of reason.

Complete the activities below in this order:

- Module 7: Lecture Materials -- Locating your Data
- Module 7: Lecture Materials -- Installing your Data
- Module 7: Lecture Materials -- Verifying your Data

After completing these activities, show your end result and walk through what you have and how you got there.

Include the following.

## Initial Data Source

### IMDb Non-Commercial Datasets

Subsets of IMDb data are available for access to customers for personal and non-commercial use.

- Link: <https://datasets.imdbws.com/>

## Data Format and Row/Column Counts

The downloaded format was tab-separated values (TSV). There are seven tables:

- akas - 8 columns, 55,165,257 rows
- titles - 10 columns, 12,301,237 rows
- crew - 3 columns, 12,301,237 rows
- episode - 4 columns, 9,403,918 rows
- principals - 6 columns, 97,893,697 rows
- ratings - 3 columns, 1,637,877 rows
- names - 6 columns, 15,103,363 rows

## Data Dictionary Example

### titles table (`\d+ titles`)

```text
imdb-# \d+ titles
                                              Table "public.titles"
     Column      |  Type   | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
-----------------+---------+-----------+----------+---------+----------+-------------+--------------+-------------
 tconst          | text    |           | not null |         | extended |             |              |
 title_type      | text    |           |          |         | extended |             |              |
 primary_title   | text    |           |          |         | extended |             |              |
 original_title  | text    |           |          |         | extended |             |              |
 is_adult        | integer |           |          |         | plain    |             |              |
 start_year      | integer |           |          |         | plain    |             |              |
 end_year        | integer |           |          |         | plain    |             |              |
 runtime_minutes | integer |           |          |         | plain    |             |              |
 genres          | text    |           |          |         | extended |             |              |
 year_start      | date    |           |          |         | plain    |             |              |
Indexes:
    "title_basics_pkey" PRIMARY KEY, btree (tconst)
    "idx_title_basics_start_year" btree (start_year)
Check constraints:
    "titles_year_valid" CHECK (start_year >= 1 AND start_year <= 9999)
Referenced by:
    TABLE "akas" CONSTRAINT "title_akas_title_id_fkey" FOREIGN KEY (title_id) REFERENCES titles(tconst)
    TABLE "crew" CONSTRAINT "title_crew_tconst_fkey" FOREIGN KEY (tconst) REFERENCES titles(tconst)
    TABLE "episode" CONSTRAINT "title_episode_parent_tconst_fkey" FOREIGN KEY (parent_tconst) REFERENCES titles(tconst)
    TABLE "episode" CONSTRAINT "title_episode_tconst_fkey" FOREIGN KEY (tconst) REFERENCES titles(tconst)
    TABLE "principals" CONSTRAINT "title_principals_tconst_fkey" FOREIGN KEY (tconst) REFERENCES titles(tconst)
    TABLE "ratings" CONSTRAINT "title_ratings_tconst_fkey" FOREIGN KEY (tconst) REFERENCES titles(tconst)
Not-null constraints:
    "title_basics_tconst_not_null" NOT NULL "tconst"
Access method: heap
```

## Obstacles and Transformations

I used IMDb data in TSV format, which was challenging to work with at this scale. I could use Excel to open five of the seven files, and only a portion of the rows from each file. I ended up using Python and ChatGPT to extract and load the data into a usable format.

The project required at least one date data type, and IMDb data did not provide one directly. I added a new column to the `titles` table called `year_start` as a date type, using January 1 for each existing start year.

### SQL used

```sql
BEGIN;

-- 1) Add the new date column
ALTER TABLE titles
ADD COLUMN year_start date;

-- 2) Populate it with Jan 1 of that year
UPDATE titles
SET year_start = make_date(start_year, 1, 1)
WHERE start_year IS NOT NULL
  AND start_year BETWEEN 1 AND 9999;

-- (Optional) Enforce valid year range going forward
ALTER TABLE titles
ADD CONSTRAINT titles_year_valid CHECK (start_year BETWEEN 1 AND 9999);

COMMIT;
```

## Table Structures (with Data Types)

### akas

![akas](screenshots/structure_akas.png)

- <PRIMARY KEY> titleId (string) - a tconst, an alphanumeric unique identifier of the title
- <PRIMARY KEY> ordering (integer) - a number to uniquely identify rows for a given titleId
- title (string) - the localized title
- region (string) - the region for this version of the title
- language (string) - the language of the title
- types (array) - enumerated set of attributes for this alternative title; one or more of: "alternative", "dvd", "festival", "tv", "video", "working", "original", "imdbDisplay" (new values may be added)
- attributes (array) - additional terms to describe this alternative title, not enumerated
- isOriginalTitle (boolean) - 0: not original title; 1: original title

### titles

![titles](screenshots/structure_titles.png)

- <PRIMARY KEY> tconst (string) - alphanumeric unique identifier of the title
- titleType (string) - type/format of the title (for example: movie, short, tvseries, tvepisode, video)
- primaryTitle (string) - the more popular title / title used on promotional materials at release
- originalTitle (string) - original title in the original language
- isAdult (boolean) - 0: non-adult title; 1: adult title
- startYear (YYYY) - release year of a title; for TV series, the series start year
- endYear (YYYY) - TV series end year; '\N' for all other title types
- runtimeMinutes - primary runtime of the title, in minutes
- genres (string array) - up to three genres associated with the title
- year_start (date) - added field to provide a date-type value

### crew

![crew](screenshots/structure_crew.png)

- <PRIMARY KEY> tconst (string) - alphanumeric unique identifier of the title
- directors (array of nconsts) - director(s) of the given title
- writers (array of nconsts) - writer(s) of the given title

### episode

![episode](screenshots/structure_episode.png)

- <PRIMARY KEY> tconst (string) - alphanumeric identifier of episode
- parentTconst (string) - alphanumeric identifier of the parent TV series
- seasonNumber (integer) - season number the episode belongs to
- episodeNumber (integer) - episode number of the tconst in the TV series

### principals

![principals](screenshots/structure_principals.png)

- <PRIMARY KEY> tconst (string) - alphanumeric unique identifier of the title
- <PRIMARY KEY> ordering (integer) - a number to uniquely identify rows for a given titleId
- nconst (string) - alphanumeric unique identifier of the name/person
- category (string) - the category of job that person was in
- job (string) - the specific job title if applicable, else '\N'
- characters (string) - the name of the character played if applicable, else '\N'

### ratings

![ratings](screenshots/structure_ratings.png)

- <PRIMARY KEY> tconst (string) - alphanumeric unique identifier of the title
- averageRating - weighted average of all individual user ratings
- numVotes - number of votes the title has received

### names

![names](screenshots/structure_names.png)

- <PRIMARY KEY> nconst (string) - alphanumeric unique identifier of the name/person
- primaryName (string) - name by which the person is most often credited
- birthYear - in YYYY format
- deathYear - in YYYY format if applicable, else '\N'
- primaryProfession (array of strings) - top 3 professions of the person
- knownForTitles (array of tconsts) - titles the person is known for

## Select * from Each Table

- akas - ![select_akas](screenshots/select20_akas.png)
- titles - ![select_titles](screenshots/select20_titles.png)
- crew - ![select_crew](screenshots/select20_crew.png)
- episode - ![select_episode](screenshots/select20_episode.png)
- principals - ![select_principals](screenshots/select20_principals.png)
- ratings - ![select_ratings](screenshots/select20_ratings.png)
- names - ![select_names](screenshots/select20_names.png)

## Interesting Queries

Include:

- At least one join
- At least one query that groups and aggregates data

### Join: Top 20 movies of all time

(Shawshank Redemption is one of my top 3 favorites.)

![TOP20MOVIES](screenshots/query_join_top20movies.png)

```sql
SELECT t.primary_title, t.start_year, r.average_rating, r.num_votes
FROM titles t
JOIN ratings r ON r.tconst = t.tconst
WHERE t.title_type = 'movie'
  AND r.num_votes >= 50000
ORDER BY r.average_rating DESC
LIMIT 20;
```

### Group by: Favorite genres

![FAV_GENRES](screenshots/query_groupby_genre.png)

```sql
SELECT UNNEST(string_to_array(genres, '|')) AS genre,
       COUNT(*) AS count
FROM titles
WHERE genres IS NOT NULL
GROUP BY genre
ORDER BY count DESC;
```

### Aggregate: Top actors and actresses

![TOP_ACTORS_ACTRESSES](screenshots/query_aggregate_topactors.png)

```sql
WITH params AS (
  SELECT 10000::int AS min_votes, 10::int AS min_titles
),
actor_titles AS (
  SELECT p.nconst, r.average_rating, r.num_votes
  FROM principals p
  JOIN ratings r ON r.tconst = p.tconst
  WHERE p.category IN ('actor', 'actress')
    AND r.num_votes >= (SELECT min_votes FROM params)
)
SELECT n.primary_name AS performer,
       COUNT(*) AS titles,
       ROUND(AVG(average_rating)::numeric, 3) AS avg_rating,
       SUM(num_votes) AS total_votes
FROM actor_titles a
JOIN names n ON n.nconst = a.nconst
GROUP BY n.primary_name
HAVING COUNT(*) >= (SELECT min_titles FROM params)
ORDER BY avg_rating DESC, total_votes DESC
LIMIT 50;
```
