# 🎵 Music Store Data Analysis Project

## 📌 Project Overview  
This project analyzes a **music store database** using **SQL in PostgreSQL (pgAdmin)**. The goal is to answer key business questions to help the store understand its sales performance, customer behavior, and most popular music genres.  

## 🎯 Objectives  
- Analyze **sales trends** and **customer behavior**.  
- Identify the **best customers** and **top-selling locations**.  
- Find the **most popular music genres** in different countries.  
- Determine the **top artists and their song counts**.  

## 🛠️ Tools & Skills Used  
- **Database**: PostgreSQL (pgAdmin)  
- **Concepts**: SQL Joins, Aggregations, CTEs, Window Functions  
- **Skills**: Data Analysis, Business Insights, Query Optimization  

---

## 🗄️ Database Schema  
Below is the **schema diagram** used for the analysis:  

![Database Schema](https://github.com/muhammed-saheer/Music-Store-Data-Analysis/blob/main/schema_diagram.png)

The dataset includes tables like:  
- **Customer**: Stores customer details.  
- **Invoice**: Tracks purchases made by customers.  
- **Invoice_Line**: Contains detailed purchase data for each invoice.  
- **Track**: Stores music track details.  
- **Genre**: Contains music genre information.  
- **Album**: Stores details about albums.  
- **Artist**: Contains artist information.  
- **Employee**: Stores employee details.  

---

## 📊 SQL Queries & Insights  

### 1️⃣ **Who is the senior most employee based on job title?**

```sql
SELECT * FROM employee
ORDER by levels desc
limit 1
```

### 2️⃣ **Which countries have the most Invoices?**

```sql
SELECT count(*) as c, billing_country
from invoice
group by billing_country
order by c desc;

```

### 3️⃣ **What are top 3 values of total invoice?**

```sql
SELECT total from invoice
order by total desc
limit 3;

```

### 4️⃣ **Which city has the best customers?**

```sql
SELECT sum(total) as invoicetotal, billing_city
from invoice
group by billing_city 
order by invoicetotal desc
limit 1;
```

### 5️⃣ **Who is the best customer?**

```sql
select customer.customer_id, customer.first_name, customer.last_name, sum(invoice.total) as total
from customer
join invoice on customer.customer_id=invoice.customer_id
group by customer.customer_id
order by total desc
limit 1;
```

### 6️⃣ **Write query to return the email, first name, last name, & Genre of all Rock Music listeners.**

```sql
select distinct email, first_name, last_name, genre.name
from customer 
join invoice on invoice.customer_id = customer.customer_id
join invoice_line on invoice_line.invoice_id = invoice.invoice_id
join track on track.track_id = invoice_line.track_id
join genre on genre.genre_id = track.genre_id
where genre.name like 'Rock'
order by email;
```

### 7️⃣ **Let's invite the artists who have written the most rock music in our dataset.**

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

### 8️⃣ **Return all the track names that have a song length longer than the average song length.**

```sql
select name, miliseconds
from track
WHERE miliseconds > (
    SELECT avg(miliseconds) AS avg_track_length
    FROM track )
ORDER BY miliseconds DESC;
```

### 9️⃣ **Find how much amount spent by each customer on artists?**

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

### 🔟 **We want to find out the most popular music Genre for each country.
SQL Query:**

```sql
WITH popular_genre AS 
(
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






