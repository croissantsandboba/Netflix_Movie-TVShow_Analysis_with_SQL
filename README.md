# 🎬 Netflix Data Dive: A SQL-Powered Binge

![Netflix Logo](https://github.com/croissantsandboba/Netflix_Movie-TVShow_Analysis_with_SQL/blob/main/logo.png)

# Description
Welcome to the ultimate binge session—but for data nerds. 🍿  
This project dives deep into Netflix’s catalog using SQL to uncover trends, insights, and hidden gems across movies and TV shows.

## 🎯 Project Goals

Here’s what we set out to explore:

- 📺 Movies vs. TV Shows: What’s dominating?
- ⭐ Top ratings across content types
- 🌍 Country-wise and year-wise content trends
- 🔎 Keyword-based and genre-based content exploration

Basically, we’re turning Netflix’s database into an interactive story powered by SQL.

--

## 📦 Dataset

Straight from Kaggle’s data vault:  
**[Netflix Movies and TV Shows Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)**

---

## 🛠️ Database Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix (
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);

Here’s a quick reference to all the SQL-powered questions tackled in this project:

---


### 1. 🍿 What’s the count of Movies vs TV Shows on Netflix?
```sql
SELECT
	type,
	COUNT(*) as total_count
FROM netflix
GROUP BY type

### 2. 🍿 What’s the most common rating for each content type?
```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1

### 3. 🍿 Which movies were released in a specific year (e.g., 2020)?
```sql
SELECT*FROM netflix
WHERE 
	type = 'Movie'
	AND
	release_year = 2020

### 4. 🍿 Which top 5 countries have the most Netflix content?
```sql
SELECT * 
FROM
(
	SELECT 
		-- country,
		TRIM(UNNEST(STRING_TO_ARRAY(country, ','))) as country,
		COUNT(*) as total_content
	FROM netflix
	GROUP BY 1
)as t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5

### 5. 🍿 What is the longest movie on Netflix?
```sql
SELECT 
	*
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC

### 6. 🍿 What content was added in the last 5 years?
```sql
SELECT
*
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'

### 7. 🍿 What content was directed by Rajiv Chilaka?
```sql
SELECT *
FROM
(

SELECT 
	*,
	UNNEST(STRING_TO_ARRAY(director, ',')) as director_name
FROM 
netflix
)
WHERE 
	director_name = 'Rajiv Chilaka'

### 8. 🍿 Which TV shows have more than 5 seasons?
```sql
SELECT *
FROM netflix
WHERE 
	TYPE = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::INT > 5

### 9. 🍿 What is the count of content by genre?
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(*) as total_content
FROM netflix
GROUP BY 1

### 10. 🍿 Which 5 years had the highest % of content released in India?
```sql
SELECT 
	country,
	release_year,
	COUNT(show_id) as total_release,
	ROUND(
		COUNT(show_id)::numeric/
								(SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100 
		,2
		)
		as avg_release
FROM netflix
WHERE country = 'India' 
GROUP BY country, 2
ORDER BY avg_release DESC 
LIMIT 5

### 11. 🍿 Which Netflix titles are Documentaries?
```sql
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'

### 12. 🍿 What content has no director listed?
```sql
SELECT*FROM netflix
WHERE 
	director IS NULL

### 13. 🍿 How many Salman Khan movies came out in the last 10 years?
```sql
SELECT * FROM netflix
WHERE 
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10

### 14. 🍿 Who are the top 10 actors in Indian-produced Netflix content?
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10

### 15. 🍿 How much content contains the keywords “kill” or “violence”?
```sql
SELECT 
    category,
	TYPE,
    COUNT(*) AS content_count
FROM (
    SELECT 
		*,
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY 1,2
ORDER BY 2

---

## 🧠 Key Takeaways

This project demonstrates how SQL can be used to explore and analyze real-world datasets through practical business questions. Here's what this analysis achieves:

- **Structured Data Exploration:** By writing targeted SQL queries, we efficiently explore a structured dataset to uncover trends, patterns, and anomalies across various dimensions such as content type, release year, country, and genre.

- **Business-Oriented Querying:** Each SQL question is designed to reflect a real business problem—helping stakeholders answer questions related to content strategy, regional performance, and user preferences.

- **Data Cleaning & Transformation:** Functions like `UNNEST`, `STRING_TO_ARRAY`, and `SPLIT_PART` are leveraged to parse multi-valued fields, enabling more accurate aggregation and classification.

- **Custom Categorization:** The project introduces logic-based categorization (e.g., content flagged by keywords), which can be extended to content tagging, sentiment filtering, or recommendation models.

- **Flexible Filtering:** The queries allow for filtering based on temporal dimensions (recent content), creators (actors/directors), or themes (documentaries, violence, etc.), giving stakeholders flexible tools for segmentation.

- **Scalable Design:** The approach and queries are modular and scalable—easily adapted to similar datasets or used as building blocks in reporting dashboards or analytics pipelines.

This analysis serves as a practical foundation for anyone looking to apply SQL in media analytics, content curation, or product decision-making using structured entertainment data.
