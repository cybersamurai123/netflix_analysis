netflix_analysis
# 📺 Netflix Content Analysis Using SQL

## 🎯 Objective

To explore, analyze, and derive insights from the Netflix content catalog using SQL. The aim is to understand content distribution by type, rating, release trends, top actors and countries, and keyword-based classification to assist business decisions and viewer insights.

---

## 🧩 Problem Statement

With thousands of titles, Netflix’s global content library is vast and diverse. This project analyzes Netflix's dataset to uncover patterns such as content type distribution, popular ratings, country-specific production, trending genres, and actor appearances. Understanding such data can help content teams decide on regional content strategies, popular genres, and audience preferences.

---

## 📊 Methodology

1. Created the `netflix` table and imported data.
2. Executed 15 business questions using SQL to uncover insights.
3. Used data transformation techniques (e.g., `UNNEST`, `SPLIT_PART`) for cleaner analysis.
4. Identified missing values and metadata gaps.
5. Derived insights from genre, cast, rating, duration, and geography.

---

## 📚 Questions & Queries

### 1. Count the number of Movies vs TV Shows.
```sql
SELECT type, COUNT(*) FROM netflix GROUP BY 1;
```

### 2. Find the most common rating for Movies and TV Shows.
```sql
WITH RatingCounts AS (
  SELECT type, rating, COUNT(*) AS rating_count
  FROM netflix GROUP BY type, rating
),
RankedRatings AS (
  SELECT *, RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
  FROM RatingCounts
)
SELECT type, rating AS most_frequent_rating FROM RankedRatings WHERE rank = 1;
```

### 3. List all movies released in a specific year (e.g., 2020).
```sql
SELECT * FROM netflix WHERE release_year = 2020;
```

### 4. Find the top 5 countries with the most content on Netflix.
```sql
SELECT * FROM (
  SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS country, COUNT(*) AS total_content
  FROM netflix GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC LIMIT 5;
```

### 5. Identify the longest movie.
```sql
SELECT * FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

### 6. Find the content added in the last 5 years.
```sql
SELECT * FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

### 7. Find all Movies/TV Shows by director 'Rajiv Chilaka'.
```sql
SELECT * FROM (
  SELECT *, UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
  FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

### 8. List all TV Shows with more than 5 seasons.
```sql
SELECT * FROM netflix
WHERE type = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

### 9. Count the number of content items in each genre.
```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

### 10. Find each year and the average number of content releases in India.
```sql
SELECT 
  country,
  release_year,
  COUNT(show_id) AS total_release,
  ROUND(
    COUNT(show_id)::numeric / 
    (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
  ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

### 11. List all movies that are Documentaries.
```sql
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

### 12. Find all content without a director.
```sql
SELECT * FROM netflix
WHERE director IS NULL;
```

### 13. Find how many movies actor 'Salman Khan' appeared in the last 10 years.
```sql
SELECT * FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

### 14. Find the Top 10 Actors who have appeared in the most movies produced in India.
```sql
SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor, COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

### 15. Categorize content based on presence of 'kill' or 'violence' keywords.
```sql
SELECT category, COUNT(*) AS content_count FROM (
  SELECT 
    CASE 
      WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
      ELSE 'Good'
    END AS category
  FROM netflix
) AS categorized_content
GROUP BY category;
```

---

## 🔍 Key Insights

- **Movies** dominate the Netflix library compared to TV Shows.
- **TV Shows** mostly receive a consistent rating like "TV-MA".
- **India** ranks high in terms of content production, especially in regional storytelling.
- **Drama**, **Documentaries**, and **International TV Shows** are common genres.
- **Actors** like Salman Khan have significant recurring roles.
- A notable number of entries **lack director information**, affecting metadata quality.
- Descriptions with **"kill"** or **"violence"** can help flag potentially sensitive content.

---

## 📅 Conclusion

This project highlights how SQL-based analysis of Netflix content can deliver powerful insights on audience preferences, geographic trends, and metadata quality. Such findings can be leveraged by content curators, marketers, and data analysts in the streaming industry.

---

## 🔭 Future Scope

- Build visual dashboards using Tableau or Power BI.
- Apply NLP to extract sentiment and themes from descriptions.
- Connect external APIs like IMDb for enhanced metadata.
- Automate keyword detection for content moderation.

---

## 👤 Author

**Sagar Tomar**  
MBA in Business Analytics  
