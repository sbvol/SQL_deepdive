

```python
USE sakila;

# 1a - Select first and last name from actor table
SELECT first_name, last_name
FROM actor;

# 1b - Select first and last name but combine result into one column - all caps
SELECT CONCAT(first_name, ' ', last_name)
AS Actor_Name
FROM actor;

# 2a - Select actors named Joe
SELECT actor_id, first_name, last_name
FROM actor
WHERE first_name = 'Joe';

# 2b - Select actors whose last name includes 'gen'
SELECT actor_id, first_name, last_name
FROM actor
WHERE last_name
LIKE '%gen%';

# 2c - Select actors whose last name includes 'il'
SELECT actor_id, first_name, last_name
FROM actor
WHERE last_name
LIKE '%li%'
ORDER BY last_name, first_name;

# 2d - Select country_id from a group of countries'
SELECT country_id, country
FROM country
WHERE country
IN ('Afghanistan', 'Bangladesh', 'China');

# 3a - Add a column to the actor table and move middle_name after first_name
ALTER TABLE actor ADD middle_name VARCHAR(50) NOT NULL;
ALTER TABLE `sakila`.`actor` 
CHANGE COLUMN `middle_name` `middle_name` VARCHAR(50) NOT NULL AFTER `first_name`;

SELECT middle_name
FROM actor;

# 3b - Change data type of middle_name to BLOB
ALTER TABLE `sakila`.`actor` 
CHANGE COLUMN `middle_name` `middle_name` BLOB NOT NULL ;

# 3c - Deleting the middle_name column
ALTER TABLE `sakila`.`actor` 
DROP COLUMN `middle_name`;

# 4a - Selecting last_name and counting the number of people who share that last_name
SELECT last_name, COUNT(*)
FROM actor
GROUP BY last_name;

# 4b - Selecting last_name that is shared by 2 or more people
SELECT last_name, COUNT(*) AS cnt
FROM actor
GROUP BY last_name
HAVING cnt >= 2;

# 4c - Changing the first name of a record in a table from GROUCHO to HARPO
SELECT first_name, last_name
FROM actor
WHERE first_name = 'GROUCHO' AND last_name = 'WILLIAMS';

UPDATE actor
SET first_name = 'HARPO'
WHERE first_name = 'GROUCHO' AND last_name = 'WILLIAMS';

SELECT first_name, last_name
FROM actor
WHERE first_name = 'HARPO' AND last_name = 'WILLIAMS';

# 4d - Changing the name back to GROUCHO
SELECT first_name, last_name
FROM actor
WHERE first_name = 'HARPO' AND last_name = 'WILLIAMS';

UPDATE actor
SET first_name = 'GROUCHO'
WHERE first_name = 'HARPO' AND last_name = 'WILLIAMS';

SELECT first_name, last_name
FROM actor
WHERE first_name = 'GROUCHO' AND last_name = 'WILLIAMS';

# 5a - Showing the schema of the address table
DESCRIBE sakila.address;

# If the schema for address table needed to be recreated again
CREATE TABLE address(
	address_id INTEGER(5) AUTO_INCREMENT NOT NULL,
    address VARCHAR(50) NOT NULL,
    address2 VARCHAR(50),
    district VARCHAR(20),
    city_id INTEGER(5),
    postal_code VARCHAR(10),
    phone VARCHAR(20),
    last_update TIMESTAMP NOT NULL,
    primary key (address_id));

# 6a - Select first and last name plus street address using staff and address table
SELECT *
FROM staff;

SELECT address_id, first_name, last_name
FROM staff;

SELECT first_name, last_name, staff.address_id, address.address
FROM staff
JOIN address
WHERE staff.address_id = address.address_id;


# 6b - Determine the total amount rung up by each staff member
USE sakila;

SELECT *
FROM staff;

SELECT staff_id, SUM(payment.amount)
FROM payment
GROUP BY staff_id;
#WHERE payment.staff_id = 1
#OR payment.staff_id=2;

# Sum the total payments by staff
SELECT DISTINCT staff.staff_id, staff.first_name, staff.last_name
FROM staff;

SELECT DISTINCT payment.staff_id, SUM(payment.amount) AS Total
FROM staff 
INNER JOIN payment
WHERE staff.staff_id
IN (
	SELECT payment.staff_id
	FROM payment
)
GROUP BY payment.staff_id;

# 6c - List each film and the number of actors in that film
SELECT DISTINCT title, COUNT(film_actor.actor_id) as Total_Actors
FROM film
JOIN film_actor
WHERE film.film_id = film_actor.film_id
GROUP BY film_actor.film_id;

# 6d - Count the number of Hunchback Impossible in inventory
SELECT COUNT(inventory_id) AS Total
FROM inventory
WHERE inventory.film_id
IN (
	SELECT film.film_id
	FROM film
	WHERE film.title = 'Hunchback Impossible'
    );

# 6e - Determine the amount paid by customer - listed by last name from A-Z
SELECT customer.first_name, customer.last_name, COUNT(payment.amount) AS total
FROM customer
JOIN payment
WHERE customer.customer_id = payment.customer_id
GROUP BY payment.customer_id
ORDER BY customer.last_name ASC;

# 7a - Use subqueries to display the titles of movies starting with the letters K and Q whose language is English.
SELECT film.title
FROM film
WHERE film.title LIKE 'K%' or 'Q%' AND film.language_id
IN (
	SELECT language.language_id
	FROM language
	WHERE name = 'English'
    );

# 7b - Use subqueries to display all actors who appear in the film Alone Trip
SELECT first_name, last_name
FROM actor
WHERE actor_id
IN (
	SELECT actor_id
	FROM film_actor
	WHERE film_id
		IN (
			SELECT film_id
			FROM film
			WHERE title = 'Alone Trip'
			)
	);

# 7c - Need the names and email addresses of all Canadian customers
SELECT customer.first_name, customer.last_name, customer.email
FROM customer
WHERE customer.address_id
IN (		
	SELECT address.address_id
	FROM address
	WHERE address.address_id
		IN (
			SELECT city.city_id
			FROM city
			WHERE city.city_id
				IN (
					SELECT country_id
					FROM country
					WHERE country.country = 'CANADA'
					)
			)
	);

# 7d - Identify all movies categorized as famiy films
SELECT title
FROM film
WHERE film.film_id
IN (
	SELECT film_category.film_id
	FROM film_category
	WHERE film_category.category_id
	IN (
		SELECT category.category_id
		FROM category
		WHERE category.name = 'Family'
		)
	);

# 7e - Display the most frequently rented movies in descending order
SELECT f.title, i.film_id, count(r.inventory_id) as total
FROM rental AS r, inventory AS i, film AS f
WHERE r.inventory_id = i.inventory_id 
AND f.film_id = i.film_id
GROUP BY r.inventory_id
ORDER BY total DESC;

# 7f - Display how much business, in dollars, each store brought in.
USE sakila;

SELECT sta.store_id, a.address, c.city, SUM(p.amount) as total
FROM staff AS sta, payment AS p, address AS a, city AS c
WHERE sta.staff_id = p.staff_id
AND sta.address_id = a.address_id
AND c.city_id = a.city_id
GROUP BY sta.store_id;

# 7g - Display for each store its store ID, city, and country. I also added address.
SELECT sta.store_id, a.address, c.city, co.country
FROM staff AS sta, country AS co, address AS a, city AS c
WHERE sta.address_id = a.address_id
AND c.city_id = a.city_id
AND c.country_id = co.country_id;

# 7h - List the top five genres in gross revenue in descending order (category, film_category, inventory, payment, and rental).
SELECT ca.category_id, ca.name, SUM(p.amount) as total
FROM staff AS sta, payment AS p, address AS a, city AS c, category AS ca, rental AS r, inventory AS i, film_category as fi
WHERE sta.staff_id = p.staff_id
AND sta.address_id = a.address_id
AND c.city_id = a.city_id
AND p.rental_id = r.rental_id
AND r.inventory_id = i.inventory_id
AND i.film_id = fi.film_id
AND fi.category_id = ca.category_id
GROUP BY ca.name
ORDER BY total DESC
LIMIT 5;

# 8a - Create a view of solution 7h
CREATE VIEW exec_view
AS (
	SELECT ca.category_id, ca.name, SUM(p.amount) as total
	FROM staff AS sta, payment AS p, address AS a, city AS c, category AS ca, rental AS r, inventory AS i, film_category as fi
	WHERE sta.staff_id = p.staff_id
	AND sta.address_id = a.address_id
	AND c.city_id = a.city_id
	AND p.rental_id = r.rental_id
	AND r.inventory_id = i.inventory_id
	AND i.film_id = fi.film_id
	AND fi.category_id = ca.category_id
	GROUP BY ca.name
	ORDER BY total DESC
    );

# 8b - Display the view created and use the view to output
SHOW CREATE VIEW exec_view;
SELECT *
FROM exec_view;

# 8c - Deleting a view
DROP VIEW IF EXISTS exec_view;
```
