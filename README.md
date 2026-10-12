# Netflix Movies and TV Shows Data Analysis Using SQL

**A SQL-based data analytics project exploring Netflix's content catalog, ratings, release trends, genres, and country-wise distribution.**

## Project Overview

This project analyzes the Netflix Movies and TV Shows dataset using MySQL 8.0 to answer business-oriented questions and explore patterns in the available content.

The analysis focuses on content distribution, ratings, release years, countries, genres, directors, and actors. SQL queries are used to extract, filter, transform, and aggregate data into meaningful insights.

This project demonstrates practical SQL skills relevant to entry-level Data Analyst roles.

## Project Objectives

- Analyze the distribution of movies and TV shows.
- Identify the most common ratings for each content type.
- Explore content by release year, country, and duration.
- Analyze genre distribution and director information.
- Identify actors appearing in titles associated with India.
- Examine content descriptions using keyword-based analysis.
- Practice SQL aggregation, CTEs, window functions, date handling, and string manipulation.

## Dataset

- **Source:** Kaggle
- **Dataset:** Netflix Movies and TV Shows
- **Link:** [Download the dataset from Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows)

The dataset includes title information such as content type, director, cast, country, date added, release year, rating, duration, genre categories, and description.

*Note: The analysis is based on the provided dataset snapshot and does not represent Netflix's live catalog.*

## Tools and Technologies

| Tool / Technology | Purpose |
|---|---|
| MySQL 8.0 | Database querying and analysis |
| SQL | Data filtering, aggregation, and transformation |
| MySQL Workbench | SQL query execution and result inspection |
| Kaggle | Dataset source |

## Database Schema

```sql
DROP TABLE IF EXISTS netflix;

CREATE TABLE netflix (
    show_id VARCHAR(5),
    type VARCHAR(10),
    title VARCHAR(250),
    director VARCHAR(550),
    casts VARCHAR(1050),
    country VARCHAR(550),
    date_added VARCHAR(55),
    release_year INT,
    rating VARCHAR(15),
    duration VARCHAR(15),
    listed_in VARCHAR(250),
    description VARCHAR(550)
);
```

## Business Problems and SQL Solutions

### 1. Count Movies vs. TV Shows

**Objective:** Compare the number of movies and TV shows in the dataset.

```sql
SELECT
    type,
    COUNT(*) AS total_titles
FROM netflix
GROUP BY type
ORDER BY total_titles DESC;
```

### 2. Find the Most Common Rating for Movies and TV Shows

**Objective:** Identify the most frequently occurring rating for each content type.

```sql
WITH RatingCounts AS (
    SELECT
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    WHERE rating IS NOT NULL
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT
        type,
        rating,
        rating_count,
        RANK() OVER (
            PARTITION BY type
            ORDER BY rating_count DESC
        ) AS rating_rank
    FROM RatingCounts
)
SELECT
    type,
    rating AS most_frequent_rating,
    rating_count
FROM RankedRatings
WHERE rating_rank = 1;
```

### 3. List Movies Released in 2020

**Objective:** Retrieve movies released in a selected year.

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND release_year = 2020;
```

### 4. Find the Top 5 Countries by Number of Titles

**Objective:** Identify the countries associated with the largest number of titles.

```sql
WITH CountryData AS (
    SELECT
        TRIM(j.country_name) AS country_name
    FROM netflix AS n
    CROSS JOIN JSON_TABLE(
        CONCAT(
            '["',
            REPLACE(n.country, ', ', '","'),
            '"]'
        ),
        '$[*]' COLUMNS (
            country_name VARCHAR(100) PATH '$'
        )
    ) AS j
    WHERE n.country IS NOT NULL
)
SELECT
    country_name,
    COUNT(*) AS total_titles
FROM CountryData
WHERE country_name <> ''
GROUP BY country_name
ORDER BY total_titles DESC
LIMIT 5;
```

### 5. Identify the Longest Movie

**Objective:** Find the movie with the greatest running time.

```sql
SELECT
    title,
    duration
FROM netflix
WHERE type = 'Movie'
  AND duration REGEXP '^[0-9]+ min$'
ORDER BY CAST(
    SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED
) DESC
LIMIT 1;
```

### 6. Find Content Added in the Last 5 Years

**Objective:** Retrieve titles added during the five years preceding the current date.

```sql
SELECT
    show_id,
    title,
    type,
    date_added
FROM netflix
WHERE STR_TO_DATE(
    TRIM(date_added),
    '%M %e, %Y'
) BETWEEN DATE_SUB(CURDATE(), INTERVAL 5 YEAR)
      AND CURDATE()
ORDER BY STR_TO_DATE(
    TRIM(date_added),
    '%M %e, %Y'
) DESC;
```

### 7. Find Titles Directed by Rajiv Chilaka

**Objective:** Identify movies and TV shows associated with the selected director.

```sql
WITH DirectorData AS (
    SELECT
        n.show_id,
        n.title,
        n.type,
        TRIM(j.director_name) AS director_name
    FROM netflix AS n
    CROSS JOIN JSON_TABLE(
        CONCAT(
            '["',
            REPLACE(n.director, ', ', '","'),
            '"]'
        ),
        '$[*]' COLUMNS (
            director_name VARCHAR(200) PATH '$'
        )
    ) AS j
    WHERE n.director IS NOT NULL
)
SELECT DISTINCT
    show_id,
    title,
    type,
    director_name
FROM DirectorData
WHERE director_name = 'Rajiv Chilaka';
```

### 8. Find TV Shows with More Than 5 Seasons

**Objective:** Identify TV shows with a season count greater than five.

```sql
SELECT
    title,
    duration
FROM netflix
WHERE type = 'TV Show'
  AND duration REGEXP '^[0-9]+ Seasons?$'
  AND CAST(
      SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED
  ) > 5
ORDER BY CAST(
    SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED
) DESC;
```

### 9. Count Titles in Each Genre

**Objective:** Analyze how titles are distributed across genre categories.

```sql
WITH GenreData AS (
    SELECT
        TRIM(j.genre_name) AS genre_name
    FROM netflix AS n
    CROSS JOIN JSON_TABLE(
        CONCAT(
            '["',
            REPLACE(n.listed_in, ', ', '","'),
            '"]'
        ),
        '$[*]' COLUMNS (
            genre_name VARCHAR(150) PATH '$'
        )
    ) AS j
    WHERE n.listed_in IS NOT NULL
)
SELECT
    genre_name,
    COUNT(*) AS total_titles
FROM GenreData
WHERE genre_name <> ''
GROUP BY genre_name
ORDER BY total_titles DESC;
```

### 10. Find the Top 5 Release Years for India-Associated Titles

**Objective:** Identify the five release years with the highest number of titles associated with India.

```sql
SELECT
    release_year,
    COUNT(*) AS total_releases
FROM netflix
WHERE country LIKE '%India%'
  AND release_year IS NOT NULL
GROUP BY release_year
ORDER BY total_releases DESC, release_year DESC
LIMIT 5;
```

*This query ranks years by title count, rather than calculating an average release percentage.*

### 11. Find Movies Classified as Documentaries

**Objective:** Retrieve movies whose genre information includes Documentaries.

```sql
SELECT
    show_id,
    title,
    release_year,
    listed_in
FROM netflix
WHERE type = 'Movie'
  AND listed_in LIKE '%Documentaries%';
```

### 12. Find Titles Without Director Information

**Objective:** Identify records with missing director details.

```sql
SELECT
    show_id,
    title,
    type,
    director
FROM netflix
WHERE director IS NULL
   OR TRIM(director) = '';
```

### 13. Count Movies Featuring Salman Khan in the Last 10 Release Years

**Objective:** Count movies featuring Salman Khan within the current and previous nine calendar years.

```sql
SELECT
    COUNT(*) AS total_movies
FROM netflix
WHERE type = 'Movie'
  AND casts LIKE '%Salman Khan%'
  AND release_year BETWEEN YEAR(CURDATE()) - 9
                       AND YEAR(CURDATE());
```

### 14. Find the Top 10 Actors in India-Associated Titles

**Objective:** Rank actors by the number of distinct titles associated with India.

```sql
WITH IndianTitles AS (
    SELECT
        show_id,
        casts
    FROM netflix
    WHERE country LIKE '%India%'
      AND casts IS NOT NULL
),
ActorData AS (
    SELECT
        TRIM(j.actor_name) AS actor_name,
        it.show_id
    FROM IndianTitles AS it
    CROSS JOIN JSON_TABLE(
        CONCAT(
            '["',
            REPLACE(it.casts, ', ', '","'),
            '"]'
        ),
        '$[*]' COLUMNS (
            actor_name VARCHAR(200) PATH '$'
        )
    ) AS j
)
SELECT
    actor_name,
    COUNT(DISTINCT show_id) AS total_titles
FROM ActorData
WHERE actor_name <> ''
GROUP BY actor_name
ORDER BY total_titles DESC
LIMIT 10;
```

*This query includes both movies and TV shows. It can be restricted to movies by adding `type = 'Movie'` to the `IndianTitles` CTE.*

### 15. Categorize Descriptions Using 'Kill' and 'Violence' Keywords

**Objective:** Group titles according to whether their descriptions contain the specified keywords.

```sql
SELECT
    CASE
        WHEN LOWER(description) LIKE '%kill%'
          OR LOWER(description) LIKE '%violence%'
        THEN 'Contains Target Keywords'
        ELSE 'Other Content'
    END AS content_category,
    COUNT(*) AS total_titles
FROM netflix
GROUP BY content_category;
```

*These categories are keyword-based labels for analysis and do not represent a judgment about the quality of a title.*

## Key Areas of Analysis

The queries explore the following analytical areas:

- **Content distribution:** Movies versus TV shows.
- **Ratings analysis:** Most frequently occurring ratings by content type.
- **Geographical analysis:** Country-wise title counts and India-associated release trends.
- **Content characteristics:** Running times, seasons, and genres.
- **People and credits:** Director and actor analysis.
- **Data quality:** Missing director information and text-based date conversion.
- **Text analysis:** Description classification using SQL conditions.

Actual numerical findings should be recorded after executing the queries against the dataset.

## Skills Demonstrated

- MySQL 8.0 and SQL query writing
- Filtering, sorting, and aggregation
- Common Table Expressions (CTEs)
- Window functions and ranking
- Date and string manipulation
- Conditional logic using `CASE`
- Pattern matching and data-quality checks
- Business-question-driven data analysis

## Project Structure

```text
netflix-sql-analysis/
├── README.md
├── data/
│   └── netflix_titles.csv
└── sql/
    ├── 01_create_table.sql
    ├── 02_data_analysis.sql
    └── 03_data_quality_checks.sql
```

The folder structure above is a suggested organization. Include only files that actually exist in your repository.

## Conclusion

This project demonstrates how SQL can be used to explore a real-world entertainment dataset, investigate content patterns, and answer structured business questions.

It provides practical experience with data aggregation, ranking, string manipulation, date conversion, and interpreting query results. The project forms part of my ongoing Data Analytics portfolio.

## Author

**Chandrajeet Chaurasiya**

MCA Student | Data Analytics

- **GitHub:** [chandrajeetchaurasiya](https://github.com/chandrajeetchaurasiya)
- **LinkedIn:** [Chandrajeet Chaurasiya](https://www.linkedin.com/in/chandrajeet-chaurasiya/)

---

*Created for learning, SQL practice, and Data Analyst portfolio development.*
