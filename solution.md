
# 🎵 Spotify SQL Data Analysis Project

This project involves exploring and analyzing a Spotify dataset using SQL to uncover trends, performance metrics, and artist insights.

---

## 📦 Schema

```sql
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```

---

## 🔍 SQL Queries and Explanations

### 1. Tracks with Over 1 Billion Streams

```sql
SELECT * FROM spotify 	
WHERE stream > 1000000000;
```

Retrieves all tracks that have been streamed more than one billion times.

---

### 2. Albums and Artists

```sql
SELECT DISTINCT(album), artist 
FROM spotify 
ORDER BY 1;
```

Returns all unique album and artist pairs.

---

### 3. Total Comments on Licensed Tracks

```sql
SELECT SUM(comments) AS total_comments 
FROM spotify 
WHERE licensed = 'true';
```

Calculates total number of comments for licensed content.

---

### 4. Tracks from Single-Type Albums

```sql
SELECT * FROM spotify 
WHERE album_type = 'single';
```

Lists tracks that belong to albums marked as "single".

---

### 5. Count of Tracks per Artist

```sql
SELECT COUNT(*) AS total_track, artist 
FROM spotify
GROUP BY artist
ORDER BY 1 DESC;
```

Gives the number of tracks associated with each artist.

---

### 6. Average Danceability by Album

```sql
SELECT album, AVG(danceability) AS avg_danceability
FROM spotify 
GROUP BY 1
ORDER BY 2 DESC;
```

Shows average danceability score for each album.

---

### 7. Top 5 High-Energy Tracks

```sql
SELECT track, MAX(energy) AS high_energy_values
FROM spotify 
GROUP BY track
ORDER BY MAX(energy) DESC
LIMIT 5;
```

Lists the five tracks with the highest energy scores.

---

### 8. Views and Likes for Official Videos

```sql
SELECT track,
       SUM(views) AS total_views,
       SUM(likes) AS total_likes 
FROM spotify
WHERE official_video = 'true'
GROUP BY 1
ORDER BY 2 DESC;
```

Shows views and likes for tracks marked as official videos.

---

### 9. Total Views per Album and Track

```sql
SELECT album, track,
       SUM(views) AS total_view
FROM spotify 
GROUP BY 1, 2
ORDER BY 3 DESC;
```

Displays total views for each track grouped by album.

---

### 10. Tracks Streamed More on Spotify than YouTube

```sql
SELECT * FROM (
    SELECT track,
           COALESCE(SUM(CASE WHEN most_played_on = 'Youtube' THEN stream END), 0) AS streamed_as_youtube,
           COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END), 0) AS streamed_as_spotify
    FROM spotify
    GROUP BY 1
) AS T1
WHERE streamed_as_youtube <> 0 AND streamed_as_spotify > streamed_as_youtube;
```

Compares streams between platforms for each track.

---

### 11. Top 3 Most-Viewed Tracks per Artist

```sql
WITH ranked_artist AS (
    SELECT 
        artist,
        track,
        SUM(views) AS total_view,
        DENSE_RANK() OVER (PARTITION BY artist ORDER BY SUM(views) DESC) AS rank 
    FROM spotify 
    GROUP BY 1, 2
)
SELECT * 
FROM ranked_artist
WHERE rank <= 3;
```

Uses window functions to find top 3 tracks per artist by view count.

---

### 12. Tracks with Liveness Above Average

```sql
SELECT track, artist, liveness 
FROM spotify 
WHERE liveness > (
    SELECT AVG(liveness) AS avg_liveness FROM spotify
);
```

Filters tracks that have a higher-than-average liveness score.

---

### 13. Energy Range in Albums (Using WITH Clause)

```sql
WITH energy AS (
    SELECT album,
           MIN(energy) AS min_energy,
           MAX(energy) AS max_energy 
    FROM spotify 
    GROUP BY 1
)
SELECT album, max_energy - min_energy AS energy_diff 
FROM energy 
ORDER BY 2 DESC;
```

Shows energy value differences within each album.

---

### 14. Tracks with High Energy-to-Liveness Ratio (> 1.2)

```sql
SELECT track, energy, liveness,
       (energy / NULLIF(liveness, 0)) AS energy_liveness_ratio
FROM spotify
WHERE (energy / NULLIF(liveness, 0)) > 1.2;
```

Calculates energy-to-liveness ratio and filters tracks where it exceeds 1.2.

---

### 15. Cumulative Likes Ordered by Views

```sql
SELECT 
    track,
    views,
    likes,
    SUM(likes) OVER (ORDER BY views) AS cumulative_likes
FROM spotify
WHERE likes <> 0;
```

Uses window function to calculate running total of likes based on views.

---

## ✅ Conclusion

This analysis provides deep insights into track performance, artist popularity, and musical traits using SQL. Techniques include filtering, aggregation, joins, and advanced window functions.
