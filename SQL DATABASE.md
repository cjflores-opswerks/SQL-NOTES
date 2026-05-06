
# ==ORDER==
```sql
###SQL Clause Order

#1 SELECT
-- Choose the columns you want to display.

#2 FROM
-- Choose the table you are getting data from.

#3 WHERE
-- Filter rows before anything is grouped.

#4 GROUP BY
-- Group rows together, usually with COUNT, AVG, SUM, MIN, or MAX.

#5 HAVING
-- Filter grouped results.

#6 ORDER BY
-- Sort the final results.

#7 LIMIT
-- Limit how many rows are returned.

SELECT neighborhood, COUNT(*) AS restaurant_count
FROM nomnom
WHERE health IS NOT NULL
GROUP BY neighborhood
HAVING COUNT(*) > 1
ORDER BY restaurant_count DESC
LIMIT 10;
```

# ==TYPES==
```SQL
INTEGER   → whole numbers
REAL      → decimal numbers
TEXT      → words/text
BLOB      → files/raw binary data
NULL      → no value

BOOLEAN   → usually 0 or 1
DATE      → usually stored as TEXT like '2026-05-04'
DATETIME  → usually stored as TEXT like '2026-05-04 14:30:00'
VARCHAR   → text, similar to TEXT in SQLite
CHAR      → short text, similar to TEXT in SQLite
DECIMAL   → decimal number, often money
NUMERIC   → number, integer or decimal
FLOAT     → decimal number, similar to REAL
DOUBLE    → decimal number, similar to REAL
```

# ==TABLE CHECKING==
```SQL
#1 List tables
.tables

#2 Show columns + types
PRAGMA table_info(<table_name>);

#3 Show table creation code
.schema <table_name>

#4 Requirements when importing and seeing tables
.mode csv
.headers on
.mode column
```

# ==MODULE 1

## ==CREATE TABLE==
```sql
-- CREATE TABLE makes a new table.
-- Columns are written inside the parentheses.

CREATE TABLE awards (
  id INTEGER,
  name TEXT,
  recipient TEXT,
  award_name TEXT
);
  
CREATE TABLE pokedex_clean AS  
SELECT  
pokemon_id,  
name,  
type1  
FROM pokedex_messy;  

-- CREATE TABLE ... AS SELECT is used to create a new table  
-- from the result of a SELECT query.  
  
-- Meaning:  
-- Run the SELECT query first.  
-- Whatever result comes out becomes a new table called pokedex_clean.
```

## ==INSERT==
```SQL
-- INSERT INTO adds a new row to a table.
-- The values must match the columns listed.

INSERT INTO awards (id, name, recipient, award_name)
VALUES (1, 'Best Song', 'Taylor Swift', 'Grammy');

-- INSERT INTO WITH TWO VALUES

INSERT INTO table_name (column1, column2, column3)
VALUES
  (value1, value2, value3),
  (value1, value2, value3);
```
## ==ALTER==
```SQL
-- ALTER TABLE changes an existing table.
-- ADD COLUMN adds a new column to the table.

ALTER TABLE awards
ADD COLUMN year INTEGER;
```

## ==UPDATE==
```SQL
-- UPDATE changes existing data in a table.
-- WHERE chooses which row or rows to update.

UPDATE awards
SET recipient = 'Beyonce'
WHERE id = 1;
```

## ==DELETE==
```SQL
-- DELETE FROM removes rows from a table.
-- WHERE chooses which row or rows to delete.

DELETE FROM awards
WHERE id = 1;
```

## ==CONSTRAINTS==
```SQL
-- Constraints are rules for data in a table.
-- PRIMARY KEY uniquely identifies each row.
-- UNIQUE prevents duplicate values.
-- NOT NULL means the value cannot be empty.
-- DEFAULT gives a value automatically if none is provided.

CREATE TABLE awards (
  id INTEGER PRIMARY KEY,
  name TEXT UNIQUE,
  recipient TEXT NOT NULL,
  award_name TEXT DEFAULT 'Grammy'
);
```


# ==MODULE 2==

## ==SELECT SPECIFIC COLUMNS==
```SQL
-- SELECT specific columns
-- Instead of using *, you can choose only the columns you need.

SELECT name, genre, year
FROM movies;
```

## ==AS==
```SQL
-- AS
-- AS gives a column or table a temporary nickname.
-- This is useful for making results easier to read.

SELECT name AS movie_title
FROM movies;
```


## ==DISTINCT==
```sql
-- DISTINCT
-- DISTINCT removes duplicate values from the results.
-- This shows each genre only once.

SELECT DISTINCT genre
FROM movies;
```

## ==WHERE==
```SQL
-- WHERE
-- WHERE filters rows based on a condition.
-- This only shows movies released after 2010.

SELECT *
FROM movies
WHERE year > 2010;
```
## ==AND==
```sql
-- AND is used when both conditions must be true.
-- This finds romance movies from 1990 to 1999.
-- Comparison operators used with the `WHERE` clause are:

-- `=` equal to
-- `!=` not equal to
-- `>` greater than
-- `<` less than
-- `>=` greater than or equal to
-- `<=` less than or equal to

SELECT *
FROM movies
WHERE year BETWEEN 1990 AND 1999
  AND genre = 'romance';
```

## ==LIKE==
```sql
-- LIKE
-- LIKE is used to search for a pattern in text.
-- % means any number of characters.
-- This finds movies with names that start with 'The'.

SELECT *
FROM movies
WHERE name LIKE 'The%';

-- LIKE with % on both sides
-- This finds movies with the word 'man' anywhere in the name.

SELECT *
FROM movies
WHERE name LIKE '%man%';

-- LIKE with _
-- The underscore _ means exactly one character.
-- This finds names like 'Cars' or 'Mars' because _ replaces one letter.

SELECT *
FROM movies
WHERE name LIKE '_ars';
```
## ==IS NULL==
```SQL
-- IS NULL
-- IS NULL checks if a value is missing or empty.
-- Use this instead of = NULL.

SELECT *
FROM movies
WHERE imdb_rating IS NULL;
```

## ==IS NOT NULL==
```SQL
-- IS NOT NULL
-- IS NOT NULL checks if a value exists.

SELECT *
FROM movies
WHERE imdb_rating IS NOT NULL;
```
#3 ==BETWEEN==
```SQL
-- BETWEEN
-- BETWEEN filters values inside a range.
-- It includes both the starting and ending values.

SELECT *
FROM movies
WHERE year BETWEEN 1990 AND 1999;

===============================================================================

-- BETWEEN includes both ends.
-- Text is compared from the first character.
-- This means name BETWEEN 'A' AND 'J' is the same as name >= 'A' AND name <= 'J'
-- So 'J' is included, but 'Jaws' is not.

SELECT *
FROM movies
WHERE name BETWEEN 'A' AND 'J';
```

## ==AND==
```sql
-- AND
-- AND means both conditions must be true.
-- This finds romance movies released from 1990 to 1999.

SELECT *
FROM movies
WHERE year BETWEEN 1990 AND 1999
  AND genre = 'romance';
```

## ==OR== 
```sql
-- OR
-- OR means at least one condition must be true.
-- This finds movies after 2014 or movies with the action genre.

SELECT *
FROM movies
WHERE year > 2014
   OR genre = 'action';
```

## ==ORDER BY== 
```SQL
-- ORDER BY ASC
-- ASC sorts from lowest to highest or oldest to newest.
-- ASC is the default, so writing it is optional.

SELECT *
FROM movies
ORDER BY year ASC;


-- ORDER BY DESC
-- DESC sorts from highest to lowest or newest to oldest.

SELECT *
FROM movies
ORDER BY year DESC;
```

## ==LIMIT==
```sql
-- LIMIT
-- LIMIT controls how many rows are shown.
-- This shows only the first 5 results.

SELECT *
FROM movies
LIMIT 5;

-- LIMIT with ORDER BY
-- This shows the 5 newest movies.

SELECT *
FROM movies
ORDER BY year DESC
LIMIT 5;
```

## ==CASE==
```SQL
-- CASE
-- CASE creates custom categories in the results.
-- It works like an if/else statement.

SELECT name,
  CASE
    WHEN imdb_rating > 8 THEN 'Great'
    WHEN imdb_rating > 6 THEN 'Good'
    ELSE 'Okay'
  END AS rating_category
FROM movies;

-- CASE explanation
-- WHEN checks a condition.
-- THEN gives the result if the condition is true.
-- ELSE gives a result if none of the conditions are true.
-- END finishes the CASE statement.
-- AS names the new column.

SELECT name,
  CASE
    WHEN year < 2000 THEN 'Old Movie'
    ELSE 'New Movie'
  END AS movie_age
FROM movies;
```

# ==MODULE 3==

## ==COUNT==
```SQL
SELECT COUNT(*)
FROM startups;

SELECT COUNT(name)
FROM startups;

-- COUNT counts rows in a table.
-- COUNT(*) counts every row.
-- COUNT(column_name) only counts rows where that column is not NULL.
-- In this example, COUNT(name) counts startups that have a name listed.
```

## ==SUM==
```SQL
SELECT SUM(valuation)
FROM startups;

-- SUM adds all values in a numeric column.
-- This example finds the total valuation of all startups.
```

## ==MAX / MIN==
```SQL
SELECT MAX(valuation)
FROM startups;

SELECT MIN(valuation)
FROM startups;

-- MAX finds the highest value in a column.
-- MIN finds the lowest value in a column.
-- These examples find the highest and lowest startup valuations.
```
## ==AVERAGE==
```SQL
SELECT AVG(valuation)
FROM startups;

-- AVG finds the average value of a numeric column.
-- This example finds the average startup valuation.
```

## ==ROUND==
```sql
SELECT ROUND(AVG(price), 2)
FROM products;

SELECT ROUND(price, 0)
FROM products;

-- ROUND shortens a decimal number.
-- The second number tells how many decimal places to show.
-- ROUND(..., 2) shows 2 decimal places.
-- ROUND(..., 0) shows no decimal places.
```

## ==GROUP BY I==
```SQL
SELECT category, COUNT(*)
FROM products
GROUP BY category;

SELECT brand, COUNT(*)
FROM products
GROUP BY brand;

-- GROUP BY groups rows that have the same value.
-- It is usually used with aggregate functions like COUNT, AVG, SUM, MAX, and MIN.
-- These examples count products by category and by brand.

So the rule in many SQL databases is:

> If a column appears in `SELECT`, and it is not inside an aggregate function, it should appear in `GROUP BY`.
```

## ==GROUP BY II==
```SQL
SELECT category, 
       price,
       AVG(downloads)
FROM fake_apps
GROUP BY 1, 2
ORDER BY 1, 2;

-- Numbers like 1, 2, 3, 4, and 5 can represent the order of columns in the SELECT line.
-- 1 = category
-- 2 = price
-- 3 = AVG(downloads)
-- GROUP BY 1, 2 means group by category and price.
-- ORDER BY 1, 2 means sort by category first, then price.
-- This shortcut makes SQL shorter, but using the column names is usually easier to read.
```

## ==HAVING==
```sql
SELECT category, COUNT(*)
FROM products
GROUP BY category
HAVING COUNT(*) > 5;

SELECT category, AVG(price)
FROM products
GROUP BY category
HAVING AVG(price) > 100;

-- HAVING filters grouped results.
-- WHERE filters rows before grouping.
-- HAVING filters groups after GROUP BY.
-- These examples only show groups that match the aggregate condition.

# WHERE vs HAVING

-- WHERE filters rows before grouping.
-- HAVING filters groups after grouping.

SELECT category, AVG(downloads)
FROM fake_apps
WHERE price > 0
GROUP BY category
HAVING AVG(downloads) > 10000;
```

## ==ORDER BY==
```SQL
SELECT category, COUNT(*)
FROM products
WHERE price IS NOT NULL
GROUP BY category
HAVING COUNT(*) >= 3
ORDER BY COUNT(*) DESC;

-- This example combines many query tools.
-- WHERE removes rows with missing prices first.
-- GROUP BY groups products by category.
-- HAVING keeps only categories with at least 3 products.
-- ORDER BY sorts the results from highest count to lowest count.
```

# ==MODULE 4==
## ==COMBINING TABLES WITH SQL / JOIN==
```SQL
SELECT *
FROM orders
JOIN customers
  ON orders.customer_id = customers.id;

-- JOIN combines rows from two tables.
-- ON tells SQL which columns should match.
-- This example combines each order with the customer who made it.


SELECT orders.order_id,
       customers.customer_name
FROM orders
JOIN customers
  ON orders.customer_id = customers.id;

-- You can choose specific columns from multiple tables.
-- Use table_name.column_name when columns may have the same name.
-- This makes it clear which table each column comes from.

SELECT
    t1.column_name,
    t2.column_name,
    t3.column_name
FROM table1 t1
JOIN table2 t2
    ON t1.common_column = t2.common_column
JOIN table3 t3
    ON t2.common_column = t3.common_column;

-- General 3 Table Join Pattern
-- #1 Select the columns you want to show
-- #2 Start from the first table and give it an alias
-- #3 Join the second table to the first table
-- #4 Join the third table to the second table
```

## ==INNER JOIN==
```sql
SELECT *
FROM orders
INNER JOIN customers
  ON orders.customer_id = customers.id;

SELECT *
FROM orders
JOIN customers
  ON orders.customer_id = customers.id;

-- INNER JOIN only returns rows that have a match in both tables.
-- JOIN by itself usually means INNER JOIN.
-- If an order does not match a customer, it will not appear.
```

## ==LEFT JOIN==
```sql
SELECT *
FROM customers
LEFT JOIN orders
  ON customers.id = orders.customer_id;

-- LEFT JOIN returns all rows from the left table.
-- It also returns matching rows from the right table.
-- If there is no match, the right table columns show NULL.
-- This example shows all customers, even customers with no orders.
```

## ==PRIMARY KEY==
```SQL
CREATE TABLE customers (
  id INTEGER PRIMARY KEY,
  name TEXT
);

CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  customer_id INTEGER,
  item TEXT
);

-- A primary key uniquely identifies each row in a table.
-- A foreign key connects one table to another table.
-- Here, customers.id is the primary key.
-- orders.customer_id is a foreign key that refers to customers.id.
```

## ==CROSS JOIN==
```SQL
SELECT shirts.shirt_color,
       pants.pants_color
FROM shirts
CROSS JOIN pants;

-- CROSS JOIN combines every row from the first table
-- with every row from the second table.
-- If shirts has 3 rows and pants has 4 rows,
-- the result will have 12 combinations.
```

## ==UNION==
```sql
SELECT name
FROM customers
UNION
SELECT name
FROM employees;

-- UNION stacks the results of two SELECT statements.
-- Both SELECT statements must have the same number of columns.
-- UNION removes duplicate rows by default.

SELECT name
FROM customers
UNION ALL
SELECT name
FROM employees;

-- UNION ALL also stacks results from two SELECT statements.
-- The difference is that UNION ALL keeps duplicates.
-- Use UNION ALL when you want every row included.
```

## ==WITH==
```sql
WITH previous_results AS (
  SELECT customer_id,
         COUNT(*) AS order_count
  FROM orders
  GROUP BY customer_id
)
SELECT customers.name,
       previous_results.order_count
FROM customers
JOIN previous_results
  ON customers.id = previous_results.customer_id;

-- WITH creates a temporary result that can be used later in the query.
-- This is also called a common table expression, or CTE.
-- It helps make long queries easier to read.
-- Here, previous_results stores the number of orders per customer.
```

## ==MODULE 4 TIPS==
```SQL
###WHERE vs HAVING
-- Use WHERE to filter rows before grouping
WHERE CAST(attack AS INTEGER) > 50

-- Use HAVING to filter groups after grouping
HAVING COUNT(*) > 2
```


# DUPLICATION
### TO AVOID DUPLICATION ASSUMING THAT THE DATA DUPLICATES WITH THE SAME COLUMNS
```SQL
SELECT *
FROM pokedex_raw
INNER JOIN pokedex_ceji
ON pokedex_raw.name = pokedex_ceji.name
WHERE pokedex_raw.pokemon_id IN (
  SELECT MIN(pokemon_id)
  FROM pokedex_raw
  GROUP BY name
);

The first query uses `select *`, so it returns **all columns** from both tables, but it also adds a `where` condition with a subquery to keep only the row with the smallest `pokemon_id` for each Pokémon name.
This is useful when you want **all available information** from both tables but still want to avoid duplicate Pokémon names, such as duplicate Pikachu rows.


```


```sql
SELECT DISTINCT
  pokedex_raw.name,
  pokedex_raw.type1,
  pokedex_raw.attack,
  pokedex_ceji.category
FROM pokedex_raw
INNER JOIN pokedex_ceji
ON pokedex_raw.name = pokedex_ceji.name;

The second query uses `select distinct`, so it only removes duplicate rows based on the specific columns you selected: `name`, `type1`, `attack`, and `category`.
This is useful when you do **not need every column** and only want a clean result showing selected information from both tables.
```

# ==EXTRAS==

# ==CAST==
# If the type is TEXT and you want to convert to INTEGER
```sql
SELECT *
FROM pokedex_raw
WHERE CAST(attack AS INTEGER) BETWEEN 50 AND 100;
```

```SQL

### Example

```sql
INSERT INTO friends (id, name, birthday)
VALUES
  (2, 'Erl', '2001-10-25'),
  (3, 'CJ', '2001-10-24');
```

# ==Editing a value==

### Format


```sql
UPDATE table_name
SET column_name = CASE
  WHEN condition THEN new_value
  WHEN condition THEN new_value
END
WHERE condition;
```
## Example

```sql
UPDATE friends
SET email = CASE
  WHEN id = 1 THEN 'storm@codecademy.com'
  WHEN id = 3 THEN 'cj.flores@academy.opswerks.com'
END
WHERE id IN (1, 3);

SELECT * FROM friends;

```

# TO CREATE NEW TABLE AND COPY THE DATA FROM THE OLD TABLE
```sql
CREATE TABLE pokedex_test_new (
  pokemon_id TEXT PRIMARY KEY,
  pokemon_name TEXT UNIQUE NOT NULL,
  pokemon_type1 TEXT NOT NULL,
  pokemon_type2 TEXT DEFAULT 'None'
);

INSERT INTO pokedex_test_new (
  pokemon_id,
  pokemon_name,
  pokemon_type1,
  pokemon_type2
)
SELECT
  pokemon_id,
  pokemon_name,
  pokemon_type1,
  pokemon_type2
FROM pokedex_test;

DROP TABLE pokedex_test;

ALTER TABLE pokedex_test_new
RENAME TO pokedex_test;

```


# TO COPY SPECIFIC DATA FROM TABLE A TO TABLE B
```sql
INSERT INTO pokedex_test (
  pokemon_id,
  name,
  type1,
  type2
)
SELECT
  pokemon_id,
  name,
  type1,
  type2
FROM pokedex_raw
WHERE pokemon_id IN (1, 9, 15, 19, 20);
```

# TO COPY A DATA AND SKIP DUPLICATED ROWS
```sql
INSERT OR IGNORE INTO pokedex_clean (
  pokemon_id,
  name,
  type1,
  type2,
  hp,
  attack,
  defense,
  evolution_stage,
  evolves_to
)
SELECT DISTINCT
  pokemon_id,
  name,
  type1,
  type2,
  hp,
  attack,
  defense,
  evolution_stage,
  evolves_to

FROM pokedex_messy;

```



# GROUP ACTIVITY 2

# CAST

```SQL
##CORRELATE CAST TO BY ORDER, WHEN THE ATTACK TYPE IS TEXT AND CONVERTING THIS TO INTEGER


SELECT *  
FROM pokedex_raw  
ORDER BY CAST(attack AS INTEGER) ASC;
```


What are the top 3 Grass type pokemon with the highest hp?

Write your answers in your own words before submitting:

1. How does filtering affect the number of rows returned?
Filtering affects the results by only showing data that you need. Filtering limits rows based on the conditions that you have set. This filters all the unnecessary data that doesn't match what you're looking for.
2. How does sorting affect how results are viewed?
Sorting the results shows a more organized table, it is easier to check the data, and view the values in either ascending or descending. Sorting organizes the rows without changing how many rows are returned.
3. What is the purpose of limiting results?
Limiting results helps a cleaner output row. Limiting controls how many rows will be returned to you.
4. How do multiple conditions affect query results?
In a multiple conditions query, this refines the searching of data with the use of logic like "AND"and "OR". This makes it flexible to look for the data that you want.


# DUPLICATION NOTES
```SQL
###Best Duplicate Query Notes

#1
-- Best general method for duplicates:
-- Use ROW_NUMBER() when you want one row per pokemon_id.

#2
-- Sample query using only 3 selected columns:

WITH cleaned AS (
    SELECT
        p.pokemon_id,
        UPPER(TRIM(p.name)) AS name,
        UPPER(TRIM(p.type1)) AS type1,
        ROW_NUMBER() OVER (
            PARTITION BY p.pokemon_id
            ORDER BY UPPER(TRIM(p.name))
        ) AS rn
    FROM pokedex_messy p
    JOIN pokemon_stats s
        ON p.pokemon_id = s.pokemon_id
       AND UPPER(TRIM(p.name)) = UPPER(TRIM(s.name))
)
SELECT
    pokemon_id,
    name,
    type1
FROM cleaned
WHERE rn = 1
ORDER BY CAST(pokemon_id AS integer);

#3
-- PARTITION BY p.pokemon_id groups rows with the same pokemon_id.

#4
-- ORDER BY UPPER(TRIM(p.name)) decides which duplicate row becomes first.

#5
-- ROW_NUMBER() gives each duplicate row a number.

-- Example:
-- pokemon_id | name      | type1 | rn
-- 1          | BULBASAUR | GRASS | 1
-- 1          | BULBASAUR | GRASS | 2

#6
-- WHERE rn = 1 keeps only the first row from each pokemon_id group.

#7
-- Example capitalization duplicate:

-- Before:
-- pokemon_id | name      | type1
-- 1          | bulbasaur | grass
-- 1          | Bulbasaur | GRASS

-- After cleaning:
-- pokemon_id | name      | type1
-- 1          | BULBASAUR | GRASS
-- 1          | BULBASAUR | GRASS

-- Final result:
-- pokemon_id | name      | type1
-- 1          | BULBASAUR | GRASS

#8
-- Example spacing duplicate:

-- Before:
-- pokemon_id | name          | type1
-- 1          | " bulbasaur " | grass
-- 1          | "Bulbasaur"   | grass

-- After UPPER(TRIM(name)):
-- both names become BULBASAUR

-- Final result:
-- pokemon_id | name      | type1
-- 1          | BULBASAUR | GRASS

#9
-- Example conflicting duplicate:

-- pokemon_id | name      | type1
-- 1          | Bulbasaur | grass
-- 1          | Bulbasaur | poison

-- This is not a simple duplicate.
-- This is a data conflict.

#10
-- In conflicting duplicates, avoid blindly using MAX().
-- MAX(type1) might choose POISON only because it sorts higher alphabetically.

#11
-- Best rule:
-- DISTINCT removes exact duplicate rows.
-- UPPER() and TRIM() clean messy text.
-- GROUP BY with MAX() collapses groups but can hide problems.
-- ROW_NUMBER() is the best general method because it keeps one complete row.
```

# COALESCE
```SQL
#1
-- EASY EXAMPLE:
-- If type2 is NULL, show 'No second type' instead.

SELECT
    name,
    type1,
    COALESCE(type2, 'No second type') AS second_type
FROM pokedex_clean;


#2
-- HARD POKEMON EXAMPLE:
-- Try to find the real type effectiveness.
-- If no matchup row is found, use 1.0 as the default.

COALESCE((
    SELECT CASE
        WHEN effectiveness = 'super effective' THEN 2.0
        WHEN effectiveness = 'not very effective' THEN 0.5
        WHEN effectiveness = 'no effect' THEN 0.0
        ELSE 1.0
    END
    FROM type_advantage
    WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type1)
      AND UPPER(defending_type) = UPPER(misty_team.type1)
    LIMIT 1
), 1.0)

```