# Analysing Netflix movies and Tv shows using PostgeSQL
![Netflix logo](https://github.com/Leonard434/netflix_database_movies_project/blob/main/Netflix-swimlane-home.webp)
## overview
This project focuses on solving movie industry problems. I chose these common problems to extract insights from dataset focusing on country, directors best actor etc to make data-oriented decisions. 
### Dataset
I extracted this dataset from Kaggle.
- **Dataset link** [Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)
### Movie dataset solutions
#### **1.Count the number of Movies vs TV Shows**
```
SELECT 
    type,
    COUNT(*)
FROM netflix_movies
GROUP BY type;
```
#### 2.Find the most common rating for movies and TV shows
```
WITH Ratings AS(
		SELECT type,
		rating,
		count(*),
		RANK() OVER(PARTITION BY type ORDER BY count(*) DESC) AS ranking
		FROM netflix_movies
		GROUP BY 1, 2
	)
SELECT
	type,
	rating
FROM Ratings
WHERE ranking = 1;
```
#### 3.List all movies released in a 2020
```
SELECT * 
FROM netflix_movies
WHERE release_year = 2020;
```
#### 4.Find the top 5 countries with the most content on Netflix
```
SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS new_country,
	COUNT(*),
	RANK() OVER(ORDER BY COUNT(*)DESC) AS ranking
FROM netflix_movies
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```
#### 5.Identify the longest movie
```
SELECT *
FROM netflix_movies
WHERE SPLIT_PART(duration, ' ', 1)::numeric =
	(SELECT
	MAX(SPLIT_PART(duration, ' ', 1)::numeric) AS mins
	FROM netflix_movies);
 ```
#### 6.Find content added in the last 5 years
```
SELECT *
FROM netflix_movies
WHERE TO_DATE(date_added, 'MONTH DD YYYY') >=
	CURRENT_DATE - INTERVAL '5 years';
```
#### 7.Find all the movies/TV shows by director 'Rajiv Chilaka'
```
SELECT *
FROM netflix_movies
WHERE director ILIKE '%Rajiv Chilaka%';
```
#### 8.List all TV shows with more than 5 seasons
```
SELECT type,
	duration
FROM netflix_movies
WHERE duration > '5 seasons'
	AND type = 'TV Show';
```
#### 9.Count the number of content items in each genre
```
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, 
	COUNT('show_id') AS total
FROM netflix_movies
GROUP BY 1
ORDER BY 2 DESC;
```
#### 10.Find each year and the average numbers of content release in India on netflix.
```
SELECT 
	UNNEST(STRING_TO_ARRAY(country,',')) as country,
	EXTRACT(YEAR FROM TO_DATE (date_added,'Month dd, yyyy')) as date,
	COUNT(show_id)::numeric /(SELECT COUNT(show_id)
		FROM netflix_movies
	WHERE country = 'India')::numeric* 100 as total_contents
FROM netflix_movies
WHERE country = 'India'
GROUP BY 1, 2
ORDER BY 2, 3 DESC;
```
#### 11.List all movies that are documentaries
```
SELECT *
FROM netflix_movies
WHERE listed_in ILIKE '%Documentaries%';
```
#### 12.Find all content without a director
```
SELECT *
FROM netflix_movies
WHERE director IS NULL;
```
#### 13.Find how many movies actor 'Salman Khan' appeared in last 10 years
```
SELECT *
FROM netflix_movies
WHERE casts ILIKE '%Salman Khan%'
	AND TO_DATE(date_added,'Month dd, yyyy') >= CURRENT_DATE - INTERVAL '10 years' 
```
#### 14.Find the top 10 actors who have appeared in the highest number of movies produced in India.
```
SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
	COUNT(*) AS total_movies
FROM netflix_movies
WHERE country ILIKE '%India%'
GROUP BY actors
ORDER BY 2 DESC
LIMIT 10;
```
#### 15.Categorize the content as bad and good content
```
WITH CTE AS (
SELECT *,
 		CASE WHEN description ILIKE '%kill%' OR
	description ILIKE '%violence%' THEN 'Bad_content'
	ELSE  'Good_content' END category
FROM netflix_movies)

SELECT category,
	COUNT(*)
FROM CTE
GROUP BY 1;
```
# Author - Leonard Thuranira
Follow me on Linkedin
- Linkedin <https:linkedin.com/in/leonard-thuranira-67785024a/>
