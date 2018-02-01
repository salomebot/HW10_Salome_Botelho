--1a.Display the first and last names of all actors from the table actor.

USE sakila;

SELECT first_name, last_name
FROM actor;
--1b. Display the first and last name of each actor in a single column in upper case letters. Name the column 
Actor Name

ALTER TABLE actor
ADD actor_name VARCHAR (50); 

UPDATE actor
SET actor_name = CONCAT(IFNULL(first_name, ''), ' ', IFNULL(last_name, ''));
--2a.You need to find the ID number, first name, and last name of an actor, 
--of whom you know only the first name, "Joe."

SELECT actor_id, first_name, last_name
FROM actor
WHERE first_name ="Joe";
--2b. Find all actors whose last name contain the letters GEN:

SELECT *
FROM actor
WHERE last_name like ("%GEN%");
--2c. Find all actors whose last names contain the letters LI. 
--This time, order the rows by last name and first name, in that order:

SELECT  last_name, first_name
FROM actor
WHERE last_name like ("%LI%")
ORDER BY last_name;
--2d.Using IN, display the country_id and country columns of the following countries: 
--Afghanistan, Bangladesh, and China:

SELECT country_id, country
FROM country 
WHERE country IN ("Afghanistan", "Bangladesh","China");
--3a. Add a middle_name column to the table actor. 
--Position it between first_name and last_name. 
--Hint: you will need to specify the data type.

ALTER TABLE actor
ADD middle_name VARCHAR (10)
AFTER first_name;
--3b. You realize that some of these actors have tremendously long last names. 
--Change the data type of the middle_name column to blobs.

ALTER TABLE actor
MODIFY COLUMN middle_name blob;
--3c. Now delete the middle_name column.

ALTER TABLE actor
DROP COLUMN middle_name;
--4a. List the last names of actors, as well as how many actors have that last name.

SELECT DISTINCT last_name, COUNT(*) AS last_name_count
FROM actor
GROUP BY last_name;
--4b. List last names of actors and the number of actors who have that last name, 
--but only select distinct last_name, count(*) as last_name_count

SELECT DISTINCT last_name, COUNT(*) AS last_name_count
FROM actor
GROUP BY last_name
HAVING COUNT(*) >=2;
--4c.Oh, no! The actor HARPO WILLIAMS was accidentally entered in the actor table as 
#GROUCHO WILLIAMS, the name of Harpo's second cousin's husband's yoga teacher.
#Write a query to fix the record.

UPDATE actor
SET first_name ="HARPO"
WHERE (first_name ="GROUCHO" AND last_name ="WILLIAMS");
--4d.In a single query, if the first name of the actor is currently HARPO, change it to GROUCHO.
--Otherwise,change the first name to MUCHO GROUCHO, as that is exactly what the actor will be 
--with the grievous error.

UPDATE actor
SET first_name = CASE
    WHEN first_name = 'Harpo' THEN 'Groucho'
    WHEN first_name = 'Groucho' THEN 'Mucho Groucho'
    ELSE first_name 
END
WHERE last_name = 'Williams';
--5a. You cannot locate the schema of the address table. 
--Which query would you use to re-create it?

SELECT *
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME='address';
--6a. Use JOIN to display the first and last names, as well as the address, of each 
--staff member. Use the tables staff and address

SELECT s.first_name, s.last_name, a.address
FROM staff s
JOIN address a
USING (address_id);
--6b.Use JOIN to display the total amount rung up by each staff member in August of 2005.
--Use tables staff and payment.

SELECT s.first_name,s.last_name, SUM(p.amount) AS total_staff_id_August_2005
FROM payment p
JOIN staff s 
USING(staff_id)
WHERE DATE_FORMAT(p.payment_date, "%M %Y") = "August 2005"
GROUP BY staff_id;

--6c.List each film and the number of actors who are listed for that film.
--Use tables film_actor and film. Use inner join

SELECT f.title, COUNT(fa.actor_id) AS total_actor_id
FROM film_actor fa
INNER JOIN film f
USING(film_id)
GROUP BY f.film_id;

--6d.How many copies of the film Hunchback Impossible exist in the inventory system?

SELECT SUM(inventory_id)
FROM inventory
WHERE film_id IN
(SELECT film_id
FROM film
WHERE title ="Hunchback Impossible");

--6e. Using the tables payment and customer and the JOIN command, 
--list the total paid by each customer. List the customers alphabetically by last name:

SELECT c.last_name, SUM(p.amount) AS total_amount_cus
FROM payment p
JOIN customer c
USING(customer_id)
GROUP BY c.customer_id
ORDER BY c.last_name;

--7a.The music of Queen and Kris Kristofferson have seen an unlikely resurgence. 
--As an unintended consequence, films starting with the letters K and Q have also soared
--in popularity. Use subqueries to display the titles of movies starting with the 
--letters K and Q whose language is English.

SELECT title
FROM film
WHERE language_id IN (
SELECT language_id
FROM language
WHERE name = "English") AND
(title LIKE( "Q%") OR title LIKE( "K%")); 
--7b.Use subqueries to display all actors who appear in the film Alone Trip.

SELECT actor_name
FROM actor
WHERE actor_id IN 
(SELECT actor_id
FROM film_actor
WHERE film_id IN
(SELECT film_id
FROM film
WHERE title= "Alone Trip"));
--7c.You want to run an email marketing campaign in Canada, for which you will need 
--the names and email addresses of all Canadian customers. Use joins to retrieve this 
--information.

SELECT c.first_name, c.last_name, c.email
FROM customer c
WHERE address_id IN (
SELECT c.address_id
FROM customer c
WHERE c.address_id IN(
SELECT a.address_id
FROM address a
JOIN city ci
USING (city_id)
WHERE a.city_id IN(
SELECT ci.city_id
FROM city ci
JOIN country co 
USING (country_id)
WHERE ci.country_id LIKE (
SELECT co.country_id
FROM country co
WHERE co.country="CANADA"))));
--7d.Sales have been lagging among young families, and you wish to target all family 
--movies for a promotion. Identify all movies categorized as famiy films.

SELECT title
FROM film
WHERE film_id IN(
SELECT film_id
FROM film_category
WHERE category_id IN (
SELECT category_id
FROM category
WHERE name = "Family"));
--7e. Display the most frequently rented movies in descending order.

SELECT f.title
FROM film f
JOIN inventory i
USING(film_id)
WHERE i.film_id IN(
SELECT i.film_id
FROM inventory i
JOIN rental r
USING(inventory_id)
WHERE i.inventory_id IN 
(SELECT r.inventory_id
FROM rental r))
GROUP BY inventory_id
ORDER BY COUNT(inventory_id) DESC;
--7f. Write a query to display how much business, in dollars, each store brought in.

SELECT st.store_id, sum(p.amount)
FROM store st ,payment p, staff s
WHERE st.store_id=s.store_id
AND s.staff_id =p.staff_id
GROUP BY p.staff_id;
--7g. Write a query to display for each store its store ID, city, and country.

SELECT st.store_id, ci.city, co.country 
FROM store st, city ci, country co, address a 
WHERE st.address_id=a.address_id
AND a.city_id=ci.city_id
AND ci.country_id=co.country_id;
--7h. List the top five genres in gross revenue in descending order. 
--(Hint: you may need to use the following tables: category, film_category, 
--inventory, payment, and rental.)

SELECT  cat.name, sum(p.amount)
FROM category cat, rental r, payment p, film_category fc, inventory i
WHERE p.rental_id=r.rental_id
AND i.inventory_id=r.inventory_id
AND fc.film_id=i.film_id
AND cat.category_id=fc.category_id
GROUP BY cat.name
ORDER BY sum(p.amount) DESC LIMIT 5;
--8a.In your new role as an executive, you would like to have an easy way of viewing the
--Top five genres by gross revenue. Use the solution from the problem above to create a view. 
--If you haven't solved 7h, you can substitute another query to create a view.

CREATE VIEW top5_genres AS
SELECT  cat.name, sum(p.amount)
FROM category cat, rental r, payment p, film_category fc, inventory i
WHERE p.rental_id=r.rental_id
AND i.inventory_id=r.inventory_id
AND fc.film_id=i.film_id
AND cat.category_id=fc.category_id
GROUP BY cat.name
ORDER BY sum(p.amount) DESC LIMIT 5;
--8b. How would you display the view that you created in 8a?

Select*
FROM top5_genres;
--8c. You find that you no longer need the view top_five_genres. 
--Write a query to delete it.

DROP VIEW top5_genres;


