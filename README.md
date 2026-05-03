# Hotstar Content Analysis — SQL Project

![SQL](https://img.shields.io/badge/Tool-SQL-blue) ![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-336791) ![Status](https://img.shields.io/badge/Status-Completed-brightgreen) ![Records](https://img.shields.io/badge/Records-6874-orange) ![Years](https://img.shields.io/badge/Years-1928--2023-red)

---

##  Project Overview

This project performs a comprehensive content analysis of **Disney+ Hotstar's streaming library** using **SQL (PostgreSQL)**. The dataset contains **6,874 titles** spanning nearly a century **(1928–2023)**, covering movies and TV shows across **37 genres** with details on age ratings, running time, seasons, and episodes.

---

##  Objective

To analyze Hotstar's content library and extract meaningful insights such as:
- What is the split between movies and TV shows?
- Which genres dominate the platform?
- How has content grown over the decades?
- What percentage of content is family-friendly vs adult?
- Which TV shows have the most seasons and episodes?

---

## Dataset Details

| Property | Value |
|---|---|
| Total Titles | 6,874 |
| Movies | 4,568 |
| TV Shows | 2,306 |
| Time Period | 1928 – 2023 |
| Total Genres | 37 |
| Top Genres | Drama, Comedy, Romance, Action, Reality |
| Age Ratings | U, U/A 7+, U/A 13+, U/A 16+, A, PG |

---

## Database Schema

### Table: `hotstar`

| Column | Data Type | Description |
|---|---|---|
| `hotstar_id` | BIGINT (PK) | Unique content identifier |
| `title` | VARCHAR(200) | Title of the movie or show |
| `description` | TEXT | Brief description of the content |
| `genre` | VARCHAR(50) | Content genre |
| `year` | INT | Release year |
| `age_rating` | VARCHAR(15) | Age certification |
| `running_time` | FLOAT | Running time in minutes (movies) |
| `seasons` | FLOAT | Number of seasons (TV shows) |
| `episodes` | FLOAT | Number of episodes (TV shows) |
| `type` | VARCHAR(10) | Content type — movie or tv |

### Schema Setup

```sql
CREATE TABLE hotstar
(
    hotstar_id   BIGINT PRIMARY KEY,
    title        VARCHAR(200),
    description  TEXT,
    genre        VARCHAR(50),
    year         INT,
    age_rating   VARCHAR(15),
    running_time FLOAT,
    seasons      FLOAT,
    episodes     FLOAT,
    type         VARCHAR(10)
);
```

---

##  Business Questions & SQL Queries

---

### Q1. Total Content by Type
**How many movies vs TV shows are available on Hotstar?**

```sql
SELECT 
    type,
    COUNT(*) AS total_content
FROM hotstar
GROUP BY type
ORDER BY total_content DESC;
```

---

### Q2. Most Popular Genres
**Which genres have the most content on Hotstar?**

```sql
SELECT 
    genre,
    COUNT(*) AS total_content,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER()::NUMERIC, 2) AS percentage
FROM hotstar
GROUP BY genre
ORDER BY total_content DESC
LIMIT 10;
```

---

### Q3. Content Added Per Year
**How has Hotstar's content library grown year by year?**

```sql
SELECT 
    year,
    COUNT(*) AS total_content,
    SUM(COUNT(*)) OVER(ORDER BY year) AS running_total
FROM hotstar
GROUP BY year
ORDER BY year;
```

---

### Q4. Age Rating Distribution
**What is the content breakdown by age rating?**

```sql
SELECT 
    age_rating,
    COUNT(*) AS total_content,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER()::NUMERIC, 2) AS percentage
FROM hotstar
GROUP BY age_rating
ORDER BY total_content DESC;
```

---

### Q5. Longest Movies on Hotstar
**What are the top 10 longest movies by running time?**

```sql
SELECT 
    title,
    genre,
    year,
    running_time AS running_time_mins
FROM hotstar
WHERE type = 'movie'
    AND running_time IS NOT NULL
ORDER BY running_time DESC
LIMIT 10;
```

---

### Q6. TV Shows with Most Seasons
**Which TV shows have the most seasons?**

```sql
SELECT 
    title,
    genre,
    seasons,
    episodes
FROM hotstar
WHERE type = 'tv'
    AND seasons IS NOT NULL
ORDER BY seasons DESC
LIMIT 10;
```

---

### Q7. Average Running Time by Genre
**Which genre has the longest average movie duration?**

```sql
SELECT 
    genre,
    COUNT(*) AS total_movies,
    ROUND(AVG(running_time)::NUMERIC, 2) AS avg_running_time_mins
FROM hotstar
WHERE type = 'movie'
    AND running_time IS NOT NULL
GROUP BY genre
ORDER BY avg_running_time_mins DESC;
```

---

### Q8. Content by Decade
**Which decade had the most content produced?**

```sql
SELECT 
    CONCAT(FLOOR(year / 10) * 10, 's') AS decade,
    COUNT(*) AS total_content,
    COUNT(CASE WHEN type = 'movie' THEN 1 END) AS movies,
    COUNT(CASE WHEN type = 'tv' THEN 1 END) AS tv_shows
FROM hotstar
WHERE year IS NOT NULL
GROUP BY 1
ORDER BY decade;
```

---

### Q9. Genre Breakdown by Content Type
**Which genres are more movie-heavy vs TV-heavy?**

```sql
SELECT 
    genre,
    COUNT(CASE WHEN type = 'movie' THEN 1 END) AS movies,
    COUNT(CASE WHEN type = 'tv' THEN 1 END) AS tv_shows,
    COUNT(*) AS total
FROM hotstar
GROUP BY genre
ORDER BY total DESC
LIMIT 10;
```

---

### Q10. Adult vs Family Friendly Content
**How much content is family friendly vs adult?**

```sql
SELECT 
    CASE 
        WHEN age_rating IN ('U', 'U/A 7+') THEN 'Family Friendly'
        WHEN age_rating = 'U/A 13+' THEN 'Teen'
        WHEN age_rating IN ('U/A 16+', 'A') THEN 'Adult'
        ELSE 'Not Rated'
    END AS content_group,
    COUNT(*) AS total_content,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER()::NUMERIC, 2) AS percentage
FROM hotstar
GROUP BY 1
ORDER BY total_content DESC;
```

---

##  Key SQL Concepts Used

- Aggregate functions: `COUNT()`, `AVG()`, `ROUND()`
- `GROUP BY` for genre, type, and age rating analysis
- Window functions: `SUM() OVER()`, `COUNT() OVER()` for percentages and running totals
- `CASE WHEN` for content group classification and decade bucketing
- `CONCAT()` and `FLOOR()` for decade labeling
- `IS NOT NULL` filtering for clean analysis
- `LIMIT` for top-N queries
- PostgreSQL type casting with `::NUMERIC`

---

##  Key Findings

- Hotstar has **4,568 movies** and **2,306 TV shows** — movies dominate at ~66%
- **Drama** is the most popular genre with 2,043 titles
- Content spans nearly **100 years** — from 1928 to 2023
- Age ratings split across **6 categories** — U, U/A 7+, U/A 13+, U/A 16+, A, PG
- **Comedy, Romance, Action** are the next top genres after Drama
- Running total query tracks how the library has grown year by year

---

##  Project Structure

```
hotstar-content-analysis-sql/
│
├── hotstar_sql.sql       # All SQL queries
├── hotstar.csv           # Raw content dataset
└── README.md             # Project documentation
```

---

##  Tools Used

- **PostgreSQL** — Database & query execution
- **SQL** — Data analysis and reporting

---

## How to Run

1. Set up a PostgreSQL database
2. Run the `CREATE TABLE` statement from `hotstar_sql.sql`
3. Import `hotstar.csv` into the `hotstar` table
4. Execute the analysis queries one by one

---

##  Author

**RANU CHOUDHARY**  
Data Analyst Trainee  
📧 choudharyranu54@gmail.com  | 🔗 [[LinkedIn](https://www.linkedin.com/in/ranu-choudhary-36aa6a325?utm_source=share_via&utm_content=profile&utm_medium=member_android)] | 💻 [https://github.com/ranu-analytics/Data-Analyst-Portfolio]

---

> *This project was built to strengthen SQL skills through real-world OTT platform content data analysis.*
