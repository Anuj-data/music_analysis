# Introduction  
🚀 **Dive into the Data Job Market!** 📊  
This project focuses on analyzing a music playlist database using SQL, covering a range of questions from basic to advanced levels. By examining the dataset, you can gain insights that help the store understand its business growth and make informed decisions based on data analysis. 🌐

## Background  
The questions I aimed to answer through my SQL queries were:  
- 💼💸 **Who is the senior-most employee based on job title?**  
- 🌍📈 **Which countries have the most invoices?**  
- 💰📊 **What are the top 3 values of total invoices?**  
- 🎉🏙️ **Which city has the best customers?**  
- 🥇👤 **Who is the best customer?**  
- 🎸🎶 **Which customers are rock music listeners?**  
- 🎤🎵 **Which artists have produced the most rock music?**  
- ⏱️🎶 **What are the longest tracks?**  
- 💵👥 **How much has each customer spent on artists?**  
- 🎼🌎 **What is the most popular genre in each country?**  
- 🌍💰 **Who has spent the most on music in each country?**  

## Tools I Used  
For my analysis of the music playlist database, I utilized the following key tools:  
- **SQL:** The backbone of my analysis, allowing me to query the database and unearth critical insights.  
- **PostgreSQL:** The chosen database management system, ideal for handling the dataset.  
- **Visual Studio Code:** My go-to for executing SQL queries and managing the database.  
- **Git & GitHub:** Essential for version control and sharing the SQL scripts and analysis, ensuring collaboration and project tracking.  

## The Analysis  
Each query in this project aimed to investigate specific aspects of the music playlist database.

### 1. Senior-Most Employee  
To identify the senior-most employee based on job title:
```sql
SELECT 
    *
FROM
    employee
ORDER BY 
    levels DESC
LIMIT 
    1;
```
### 2. Countries with Most Invoices  
This query reveals which countries have the most invoices:
```sql
SELECT
    COUNT(*) AS number_of_invoices,
    billing_country 
FROM
    invoice
GROUP BY  
    billing_country
ORDER BY
    number_of_invoices DESC;
```

### 3. Top 3 Invoice Totals
To find the top 3 total invoice values:
```sql
SELECT 
    total
FROM
    invoice
ORDER BY 
    total DESC
LIMIT
    3;
```
### 4. City with Best Customers
Determining the city with the highest sum of invoice totals:
```sql
SELECT
    SUM(total) AS invoice_total,
    billing_city
FROM
    invoice
GROUP BY 
    billing_city
ORDER BY 
    invoice_total DESC
LIMIT 1;
```
### 5. Best Customer
Finding the customer who has spent the most money:
```sql
SELECT
    customer.customer_id,
    customer.first_name,
    customer.last_name,
    SUM(invoice.total) AS total
FROM
    customer
JOIN
    invoice
ON
    customer.customer_id = invoice.customer_id
GROUP BY
    customer.customer_id
ORDER BY
    total DESC 
LIMIT 1;
```
### 6. Rock Music Listeners
Identifying rock music listeners:
```sql
SELECT email, first_name, last_name
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoice_line ON invoice.invoice_id = invoice_line.invoice_id
WHERE track_id IN(
    SELECT track_id FROM track
    JOIN genre ON track.genre_id = genre.genre_id
    WHERE genre.name LIKE 'Rock'
)
ORDER BY email;
```
### 7. Top Rock Bands
Inviting the artists who have produced the most rock music:
```sql
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
### 8. Longest Tracks
Finding tracks longer than the average song length:
```sql
SELECT 
    name AS track_names,
    milliseconds
FROM
    track
WHERE milliseconds > (SELECT AVG(milliseconds) FROM track)
ORDER BY 
    milliseconds DESC;
```
### 9. Customer Spending on Artists
How much each customer has spent on artists:
```sql
WITH best_selling_artist AS (
    SELECT artist.artist_id AS artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
    FROM invoice_line
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN album ON album.album_id = track.album_id
    JOIN artist ON artist.artist_id = album.artist_id
    GROUP BY 1
    ORDER BY 3 DESC
    LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1, 2, 3, 4
ORDER BY 5 DESC;
```
### 10. Most Popular Genre by Country
Finding the most popular music genre for each country:

```sql
WITH popular_genre AS (
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
    ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
    JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
    JOIN customer ON customer.customer_id = invoice.customer_id
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN genre ON genre.genre_id = track.genre_id
    GROUP BY 2, 3, 4
    ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1;
```

### 11. Top Spending Customer by Country
Determining the customer that has spent the most on music in each country:
```sql
WITH RECURSIVE 
    customer_with_country AS (
        SELECT customer.customer_id, first_name, last_name, billing_country, SUM(total) AS total_spending
        FROM invoice
        JOIN customer ON customer.customer_id = invoice.customer_id
        GROUP BY 1, 2, 3, 4
    ),
    country_max_spending AS (
        SELECT billing_country, MAX(total_spending) AS max_spending
        FROM customer_with_country
        GROUP BY billing_country
    )
SELECT cc.billing_country, cc.total_spending, cc.first_name, cc.last_name, cc.customer_id
FROM customer_with_country cc
JOIN country_max_spending ms
ON cc.billing_country = ms.billing_country
WHERE cc.total_spending = ms.max_spending
ORDER BY 1;
```
## What I Learned
This project not only honed my SQL skills but also enriched my understanding of data analysis. Key takeaways include:

- 🎓 Mastering complex queries and data aggregation.
- 📊 Gaining insights into customer behavior and spending patterns.
- 💡 Understanding the importance of data-driven decision-making.

## Conclusions
### Insights on Employee and Customer Behavior
The analysis provides a comprehensive overview of employee hierarchy and customer spending trends.

### Data-Driven Decisions
Results can inform business strategies for promotions, targeting top customers, and enhancing service offerings.

## Closing Thoughts
This SQL project has been an enriching experience, providing valuable insights into employee and invoice data while strengthening my analytical skills. 🌟

Feel free to explore the code and results! If you have any questions, don't hesitate to reach out. 🤝
