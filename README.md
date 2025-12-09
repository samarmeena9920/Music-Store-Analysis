
# 🎵 Music Store SQL Analysis Project

## Project Overview

**Project Title**: Music Store Data Analysis
**Database**: `music_store_db`

This project demonstrates advanced SQL skills typically used by data analysts and data engineers to analyze digital music store data. You will work with multiple relational tables such as customers, employees, invoices, tracks, artists, and genres—similar to real-world datasets used in the music streaming industry.

The goal of this project is to perform **database setup, data cleaning, exploratory analysis**, and to answer **business-driven analytical questions** using SQL.
This project is ideal for learners who want hands-on experience with relational databases and analytical SQL.

---

## 🎯 Objectives

1. **Set Up the Music Store Database**
   Create and populate the database schema required for music store operations.

2. **Data Cleaning & Type Casting**
   Standardize data types (especially numeric fields stored as text) and handle any inconsistencies.

3. **Exploratory Data Analysis (EDA)**
   Build foundational understanding of dataset size, relationships, and structural integrity.

4. **Business Analysis Using SQL**
   Write SQL queries to solve real-world business problems such as identifying top customers, most popular genres, most valuable artists, and more.

---

## 📂 Project Structure

### 1. Database Setup

The music store database is created with multiple interconnected tables such as employees, customers, tracks, artists, invoices, invoice lines, playlists, albums, media types, and genres.

#### **Fixing Data Types (Example)**

Some numeric fields in source data are stored as text and must be converted:

```sql
ALTER TABLE assets
ALTER COLUMN asset_no TYPE INT 
USING asset_no::INT;
```

#### **Table Creation Queries**

All table definitions used in the project:

```sql
CREATE TABLE employee(
    employee_id VARCHAR(50) PRIMARY KEY,
    last_name CHAR(50),
    first_name CHAR(50),
    title VARCHAR(50),
    reports_to VARCHAR(30),
    levels VARCHAR(10),
    birthdate TIMESTAMP,
    hire_date TIMESTAMP,
    address VARCHAR(120),
    city VARCHAR(50),
    state VARCHAR(50),
    country VARCHAR(30),
    postal_code VARCHAR(30),
    phone VARCHAR(30),
    fax VARCHAR(30),
    email VARCHAR(30)
);

CREATE TABLE customer(
    customer_id VARCHAR(30) PRIMARY KEY,
    first_name CHAR(30),
    last_name CHAR(30),
    company VARCHAR(30),
    address VARCHAR(30),
    city VARCHAR(30),
    state VARCHAR(30),
    country VARCHAR(30),
    postal_code INT8,
    phone INT,
    fax INT,
    email VARCHAR(30),
    support_rep_id VARCHAR(30)
);

CREATE TABLE invoice(
    invoice_id VARCHAR(30) PRIMARY KEY,
    customer_id VARCHAR(30),
    invoice_date TIMESTAMP,
    billing_address VARCHAR(120),
    billing_city VARCHAR(30),
    billing_state VARCHAR(30),
    billing_country VARCHAR(30),
    billing_postal VARCHAR(30),
    total FLOAT8
);

CREATE TABLE invoice_line(
    invoice_line_id VARCHAR(50) PRIMARY KEY,
    invoice_id VARCHAR(30),
    track_id VARCHAR(30),
    unit_price VARCHAR(30),
    quantity VARCHAR(30)
);

CREATE TABLE track(
    track_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(30),
    album_id VARCHAR(30),
    media_type_id VARCHAR(30),
    genre_id VARCHAR(30),
    composer VARCHAR(30),
    milliseconds TIMESTAMP,
    bytes INT8,
    unit_price INT16
);

CREATE TABLE playlist(
    playlist_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(30)
);

CREATE TABLE playlist_track(
    playlist_id VARCHAR(50) PRIMARY KEY,
    track_id VARCHAR(50) PRIMARY KEY
);

CREATE TABLE artist(
    artist_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(30)
);

CREATE TABLE album(
    album_id VARCHAR(50) PRIMARY KEY,
    title VARCHAR(30),
    artist_id VARCHAR(30)
);

CREATE TABLE media_type(
    media_type_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(30)
);

CREATE TABLE genre(
    genre_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(30)
);
```

---

## 2. 🧼 Data Exploration & Cleaning

Before performing analysis, the following checks and cleanup steps are executed:

### ✔ **Record Count**

Count total rows in each table (e.g., employees, customers, invoices, tracks).

### ✔ **Unique Customer Count**

Determine how many distinct customers exist.

### ✔ **Category/Genre Exploration**

Identify all unique music genres, media types, and playlist categories.

### ✔ **Null and Missing Value Check**

Check for:

* Missing customer details
* Incomplete invoice rows
* Tracks without artists or albums
* NULL genre, media type, or price fields

Example cleanup operations:

```sql
SELECT *
FROM customer
WHERE first_name IS NULL 
   OR last_name IS NULL 
   OR email IS NULL;
```

You may also convert incorrectly stored numeric fields (e.g., `unit_price`, `quantity`) into proper numeric types for analysis.

---

## 3. 📊 Business Analysis (Summaries)

After preparing the dataset, analytical questions include:

* Who is the senior-most employee?
* Which country produces the most invoices?
* Who is the best customer by revenue?
* What is the most popular genre in each country?
* Which artists have produced the most rock tracks?
* Which customers spent the most money on the highest-selling artist?
---

📘 Question Set 1 — Easy
Q1: Who is the senior-most employee based on job title?
```
SELECT title, last_name, first_name 
FROM employee
ORDER BY levels DESC
LIMIT 1;
```
Q2: Which countries have the most invoices?
```
SELECT COUNT(*) AS c, billing_country 
FROM invoice
GROUP BY billing_country
ORDER BY c DESC;
```

Q3: What are the top 3 values of total invoice?
```
SELECT total 
FROM invoice
ORDER BY total DESC
LIMIT 3;
```
Q4: Which city has the best customers (highest revenue)?
```
SELECT billing_city, SUM(total) AS InvoiceTotal
FROM invoice
GROUP BY billing_city
ORDER BY InvoiceTotal DESC
LIMIT 1;
```

Q5: Who is the best customer (highest total spend)?
```
SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spending
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spending DESC
LIMIT 1;
```

📘 Question Set 2 — Moderate
```Q1: Retrieve email, first name, last name & genre of all Rock music listeners (ordered by email).
Method 1
SELECT DISTINCT email, first_name, last_name
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoiceline ON invoice.invoice_id = invoiceline.invoice_id
WHERE track_id IN (
    SELECT track_id 
    FROM track
    JOIN genre ON track.genre_id = genre.genre_id
    WHERE genre.name LIKE 'Rock'
)
ORDER BY email;
```

Method 2
```
SELECT DISTINCT email AS Email, first_name AS FirstName, last_name AS LastName, genre.name AS Name
FROM customer
JOIN invoice ON invoice.customer_id = customer.customer_id
JOIN invoiceline ON invoiceline.invoice_id = invoice.invoice_id
JOIN track ON track.track_id = invoiceline.track_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
ORDER BY email;
```
Q2: Top 10 rock artists by number of songs written
```
SELECT artist.artist_id, artist.name, COUNT(artist.artist_id) AS number_of_songs
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;
```

Q3: Tracks longer than the average song length
```
SELECT name, miliseconds
FROM track
WHERE miliseconds > (
    SELECT AVG(miliseconds) AS avg_track_length
    FROM track
)
ORDER BY miliseconds DESC;
```

📘 Question Set 3 — Advanced
Q1: Amount spent by each customer on the best-selling artist
```
WITH best_selling_artist AS (
    SELECT artist.artist_id AS artist_id, artist.name AS artist_name,
           SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
    FROM invoice_line
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN album ON album.album_id = track.album_id
    JOIN artist ON artist.artist_id = album.artist_id
    GROUP BY 1
    ORDER BY 3 DESC
    LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name,
       SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1, 2, 3, 4
ORDER BY 5 DESC;
```

Q2: Most popular music genre per country
Method 1 — Using CTE
```
WITH popular_genre AS (
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country,
           genre.name, genre.genre_id,
           ROW_NUMBER() OVER(
               PARTITION BY customer.country
               ORDER BY COUNT(invoice_line.quantity) DESC
           ) AS RowNo
    FROM invoice_line
    JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
    JOIN customer ON customer.customer_id = invoice.customer_id
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN genre ON genre.genre_id = track.genre_id
    GROUP BY 2, 3, 4
    ORDER BY 2 ASC, 1 DESC
)
SELECT *
FROM popular_genre
WHERE RowNo <= 1;
```
Method 2 — Recursive
WITH RECURSIVE
```
sales_per_country AS (
    SELECT COUNT(*) AS purchases_per_genre, customer.country,
           genre.name, genre.genre_id
    FROM invoice_line
    JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
    JOIN customer ON customer.customer_id = invoice.customer_id
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN genre ON genre.genre_id = track.genre_id
    GROUP BY 2, 3, 4
),
max_genre_per_country AS (
    SELECT MAX(purchases_per_genre) AS max_genre_number, country
    FROM sales_per_country
    GROUP BY 2
)
SELECT sales_per_country.*
FROM sales_per_country
JOIN max_genre_per_country
ON sales_per_country.country = max_genre_per_country.country
WHERE sales_per_country.purchases_per_genre = max_genre_per_country.max_genre_number;
```

Q3: Customer who spent the most in each country
Method 1 — Using CTE
```
WITH Customter_with_country AS (
    SELECT customer.customer_id, first_name, last_name, billing_country,
           SUM(total) AS total_spending,
           ROW_NUMBER() OVER(
               PARTITION BY billing_country
               ORDER BY SUM(total) DESC
           ) AS RowNo
    FROM invoice
    JOIN customer ON customer.customer_id = invoice.customer_id
    GROUP BY 1, 2, 3, 4
)
SELECT *
FROM Customter_with_country
WHERE RowNo <= 1;

Method 2 — Recursive
WITH RECURSIVE 
customter_with_country AS (
    SELECT customer.customer_id, first_name, last_name,
           billing_country, SUM(total) AS total_spending
    FROM invoice
    JOIN customer ON customer.customer_id = invoice.customer_id
    GROUP BY 1, 2, 3, 4
),
country_max_spending AS (
    SELECT billing_country, MAX(total_spending) AS max_spending
    FROM customter_with_country
    GROUP BY billing_country
)
SELECT cc.billing_country, cc.total_spending, cc.first_name,
       cc.last_name, cc.customer_id
FROM customter_with_country cc
JOIN country_max_spending ms
ON cc.billing_country = ms.billing_country
WHERE cc.total_spending = ms.max_spending
ORDER BY 1;
```
## Conclusion

- The Music Store SQL Project is a complete analytical SQL module covering:
- Database schema creation
- Data type corrections and cleaning
- Exploratory data analysis
- Intermediate to advanced SQL techniques (joins, subqueries, CTEs, window functions, recursive queries)
- Business-driven insights

### The findings contribute directly to business decisions such as:

- Understanding customer preferences
- Identifying top artists and genres
- Optimizing marketing strategies per country
- Supporting customer service via employee hierarchy insights
- Recognizing high-value customers for loyalty programs
