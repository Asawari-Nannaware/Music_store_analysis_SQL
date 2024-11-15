# Music_store_analysis_SQL
![STN Store Annual Report Dashboard](https://github.com/Asawari-Nannaware/Music_store_analysis_SQL/blob/main/MusicDatabaseSchema.png)
Here’s the entire README content in a single, easy-to-copy section:

# Music Store Database Analysis

This project explores data from a fictional music store using SQL queries. The goal is to extract insights into employee hierarchies, customer purchasing behavior, and popular music genres. The queries are divided into three sets based on complexity: Easy, Moderate, and Advanced.

## Project Overview

The database schema includes tables for:
- **Employee**: Information about store employees, including reporting structure.
- **Customer**: Customer contact information and support representatives.
- **Invoice** and **InvoiceLine**: Purchase records for each customer and individual track purchases within each invoice.
- **Track**, **Album**, **Artist**, **Genre**: Music metadata, including track details, albums, genres, and artists.
- **Playlist** and **PlaylistTrack**: Playlists and their associated tracks.

Each query in this project answers a specific business question, allowing us to gain insights into the music store's operations and customer preferences.

## Setup Instructions

1. **Clone the repository**:
   ```bash
   git clone https://github.com/yourusername/music-store-analysis.git
   cd music-store-analysis
   ```
2. **Load the database**: Import the provided SQL schema into your SQL environment (e.g., MySQL, PostgreSQL).
3. **Run the queries**: Execute the queries in the SQL script to explore insights.

## Query Descriptions

The queries are organized into three levels of complexity.

### Easy Queries

1. **Senior Most Employee**: Finds the employee with the highest-ranking job title.
   ```sql
   SELECT title, last_name, first_name FROM employee ORDER BY levels DESC LIMIT 1;
   ```
2. **Countries with the Most Invoices**: Identifies the countries with the highest number of invoices.
   ```sql
   SELECT billing_country, COUNT(*) AS invoice_count FROM invoice GROUP BY billing_country ORDER BY invoice_count DESC;
   ```
3. **Top Invoice Totals**: Displays the three highest invoice totals.
   ```sql
   SELECT total FROM invoice ORDER BY total DESC LIMIT 3;
   ```
4. **Best Customer City**: Determines the city that generated the most revenue, ideal for promotional events.
   ```sql
   SELECT billing_city, SUM(total) AS total_revenue FROM invoice GROUP BY billing_city ORDER BY total_revenue DESC LIMIT 1;
   ```
5. **Best Customer**: Finds the customer with the highest total spending.
   ```sql
   SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spending FROM customer JOIN invoice ON customer.customer_id = invoice.customer_id GROUP BY customer.customer_id ORDER BY total_spending DESC LIMIT 1;
   ```

### Moderate Queries

1. **Rock Music Listeners**: Returns details of customers who purchased Rock music, ordered by email.
   ```sql
   SELECT DISTINCT email, first_name, last_name FROM customer JOIN invoice ON customer.customer_id = invoice.customer_id JOIN invoiceline ON invoice.invoice_id = invoiceline.invoice_id WHERE track_id IN (SELECT track_id FROM track JOIN genre ON track.genre_id = genre.genre_id WHERE genre.name = 'Rock') ORDER BY email;
   ```
2. **Top Rock Artists**: Lists the top 10 artists with the most Rock tracks.
   ```sql
   SELECT artist.artist_id, artist.name, COUNT(track.track_id) AS number_of_tracks FROM track JOIN album ON album.album_id = track.album_id JOIN artist ON artist.artist_id = album.artist_id JOIN genre ON genre.genre_id = track.genre_id WHERE genre.name = 'Rock' GROUP BY artist.artist_id ORDER BY number_of_tracks DESC LIMIT 10;
   ```
3. **Longer-than-Average Tracks**: Returns tracks with a length above the average, ordered by duration.
   ```sql
   SELECT name, milliseconds FROM track WHERE milliseconds > (SELECT AVG(milliseconds) FROM track) ORDER BY milliseconds DESC;
   ```

### Advanced Queries

1. **Customer Spending on Artists**: Determines how much each customer spent on each artist.
   ```sql
   WITH best_selling_artist AS (SELECT artist.artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales FROM invoice_line JOIN track ON track.track_id = invoice_line.track_id JOIN album ON album.album_id = track.album_id JOIN artist ON artist.artist_id = album.artist_id GROUP BY artist.artist_id ORDER BY total_sales DESC LIMIT 1) SELECT customer.customer_id, customer.first_name, customer.last_name, bsa.artist_name, SUM(invoice_line.unit_price * invoice_line.quantity) AS amount_spent FROM invoice JOIN customer ON customer.customer_id = invoice.customer_id JOIN invoice_line ON invoice.invoice_id = invoice_line.invoice_id JOIN track ON track.track_id = invoice_line.track_id JOIN album ON album.album_id = track.album_id JOIN best_selling_artist bsa ON bsa.artist_id = album.artist_id GROUP BY customer.customer_id, customer.first_name, customer.last_name, bsa.artist_name ORDER BY amount_spent DESC;
   ```
2. **Most Popular Genre by Country**: Finds the top genre in each country by number of purchases.
   ```sql
   WITH popular_genre AS (SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo FROM invoice_line JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id JOIN customer ON customer.customer_id = invoice.customer_id JOIN track ON track.track_id = invoice_line.track_id JOIN genre ON genre.genre_id = track.genre_id GROUP BY customer.country, genre.name, genre.genre_id) SELECT country, name AS genre_name, purchases FROM popular_genre WHERE RowNo = 1;
   ```
3. **Top Customer by Country**: Identifies the highest-spending customer in each country.
   ```sql
   WITH customer_spending AS (SELECT customer.customer_id, first_name, last_name, billing_country, SUM(total) AS total_spent, ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo FROM invoice JOIN customer ON customer.customer_id = invoice.customer_id GROUP BY customer.customer_id, first_name, last_name, billing_country) SELECT billing_country, first_name, last_name, total_spent FROM customer_spending WHERE RowNo = 1;
   ```
