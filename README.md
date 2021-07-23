# SQL and Pivot Tables
This project runs SQL queries on the Sakila DVD movie rental database found on the  PostgreSQL [webpage](https://www.postgresqltutorial.com/postgresql-sample-database/).The data used can be downloaded as seen on the tutorial on PostgreSQL page and it is also included in the files of this repository as **"dvdrental.tar"**

An Entity relationship diagram ([ER Diagram](https://www.postgresqltutorial.com/wp-content/uploads/2018/03/printable-postgresql-sample-database-diagram.pdf)) can be downloaded here

4 Questions are asked and the queries used to answer the questions are stated below and on the **"queries.txt"** file.

This project displays use of joins, aggregations, common table expressions, and window functions. Then the results are imported into Microsoft Excel and using  pivot tables charts are built to help visualize the results. A PDF that visualizes the questions is found in the repo with the title **"sql-project.pdf"**

#### QUESTION 1
*Which Category of Family movies has the most rentals?*

    WITH t1 AS
    	(SELECT DISTINCT    f.title AS film_title, c.name AS category_name,
    					SUM(times_rented) OVER (PARTITION BY t1.film_id) AS number_times_movie_rented
    	FROM(
    		SELECT DISTINCT i.film_id, r.inventory_id, COUNT(*) OVER (PARTITION BY i.inventory_id) AS times_rented
    		FROM rental r
    		JOIN inventory i
    			ON r.inventory_id = i.inventory_id
    		ORDER BY 1) t1
    	JOIN film f
    		ON f.film_id = t1.film_id
    	JOIN film_category fc
    		ON fc.film_id = f.film_id
    	JOIN category c
    		ON c.category_id = fc.category_id
    	WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
    	ORDER BY c.name, f.title)

    SELECT category_name, SUM(number_times_movie_rented) AS number_of_times_rented
    FROM t1
    GROUP BY category_name
    ORDER BY 2 DESC


#### QUESTION
*What is the average duration of Family movie rentals by subcategory?*

    WITH t1 AS (
    	SELECT 	f.title, c.name, f.rental_duration,
    			NTILE(4) OVER (ORDER BY f.rental_duration)
    	FROM film f
    	JOIN film_category fc
    		ON f.film_id = fc.film_id
    	JOIN category c
    		ON c.category_id = fc.category_id
    	WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
    	ORDER BY 3)

    SELECT t1.name category_name, AVG(t1.rental_duration) average_rental_duration
    FROM t1
    GROUP BY 1
    ORDER BY 2 DESC

#### QUESTION 3
*How do the 2 stores rentals per month compare?*

    SELECT DISTINCT *, COUNT(*) OVER (PARTITION BY store_id, rental_month, rental_year) AS rental_orders
      FROM (
      	SELECT 	DATE_PART('month', r.rental_date) rental_month,
      			DATE_PART('year', r.rental_date) rental_year,
      			sr.store_id
      	FROM rental r
      	JOIN staff sf
      		ON sf.staff_id = r.staff_id
      	JOIN store sr
      		ON sr.store_id = sf.store_id) t1
      ORDER BY rental_year, rental_month

#### QUESTION 4
*How do the top 10 spending clients habits compare per month?*

    SELECT 	DISTINCT
    		DATE_TRUNC('month', payment_date) AS month,
    		CONCAT(first_name, ' ', last_name) AS full_name,
    		COUNT(*) OVER account_month_window AS count_per_month,
    		SUM(amount) OVER account_month_window AS payed_amount

    FROM(
    	SELECT customer_id, SUM(amount) amount_payed
    	FROM payment
    	WHERE DATE_PART('year',payment_date) = 2007
    	GROUP BY customer_id
    	ORDER BY amount_payed DESC
    	LIMIT 10) t1
    JOIN payment p
    	ON t1.customer_id = p.customer_id
    JOIN customer c
    	ON c.customer_id = p.customer_id
    WINDOW account_month_window AS (PARTITION BY DATE_TRUNC('month', payment_date), p.customer_id)
    ORDER BY  full_name, month
