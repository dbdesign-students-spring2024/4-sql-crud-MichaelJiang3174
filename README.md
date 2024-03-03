# Report of SQL CRUD

## Links to Data Files (Both part 1 and part 2)
- [Mock data restaurant](./data/MOCK_DATA%20restaurant.csv)
- [Mock data users](./data/MOCK_DATA%20users.csv)
- [Mock data message](./data/MOCK_DATA%20message.csv)
- [Mock data story](./data/MOCK_DATA%20story.csv)

## Part 1: Restaurant finder

### SQL Commands to create tables
CREATE TABLE restaurants (
   id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    price_tier TEXT NOT NULL,
    neighborhood TEXT NOT NULL,
    opening_hours TEXT NOT NULL,
    average_rating DECIMAL(2, 1) CHECK (average_rating >= 0 AND average_rating <= 5),
    good_for_kids BOOLEAN NOT NULL
);

CREATE TABLE reviews (
    id SERIAL PRIMARY KEY,
    restaurant_id INTEGER NOT NULL,
    rating DECIMAL(2, 1) NOT NULL CHECK (rating >= 0 AND rating <= 5),
    comment TEXT,
    review_date DATE NOT NULL,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(id) ON DELETE CASCADE
);

### SQL Commands to import data (restaurant)
sqlite3 SQL_CRUD.db<br>
.mode csv<br>
.import MOCK_DATA restaurant.csv restaurants<br>

## SQL queries (Restaurant finder)

### 1. Find All Cheap Restaurants in a Particular Neighborhood
SELECT * FROM restaurants
WHERE price_tier = 'Cheap'
AND neighborhood = 'Greenwich Village';

### 2. Find All Restaurants in a Particular Genre with 3 Stars or More, Ordered by the Number of Stars in Descending Order
SELECT * FROM restaurants
WHERE category = 'Italian'
AND average_rating >= 3
ORDER BY average_rating DESC;

### 3. Find All Restaurants That Are Open Now
SELECT * FROM restaurants
WHERE SUBSTR(opening_hours, 1, 5) <= strftime('%H:%M', 'now', 'localtime')
AND SUBSTR(opening_hours, 7, 5) >= strftime('%H:%M', 'now', 'localtime');

### 4. Leave a Review for a Restaurant
INSERT INTO reviews (restaurant_id, rating, comment, review_date)
VALUES (1, 5, 'Outstanding experience with delicious food!', '2024-03-01');

### 5. Delete All Restaurants That Are Not Good for Kids
DELETE FROM restaurants
WHERE good_for_kids = FALSE;

### 6. Find the Number of Restaurants in Each NYC Neighborhood
SELECT neighborhood, COUNT(*) AS num_restaurants
FROM restaurants
WHERE city = 'New York City'
GROUP BY neighborhood
ORDER BY num_restaurants DESC;

## Part 2: Social media app

### SQL code to create tables for social media
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    handle TEXT UNIQUE NOT NULL
);

CREATE TABLE posts (
    post_id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    post_type TEXT CHECK (post_type IN ('Message', 'Story')),
    content TEXT NOT NULL,
    recipient_id INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    visible BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (recipient_id) REFERENCES users(user_id)
);

### SQL Commands to import data (user, message, story)
sqlite3 SQL_CRUD.db<br>
.mode csv<br>
.import data/MOCK_DATA users.csv users<br>
.import data/MOCK_DATA message.csv posts<br>
.import data/MOCK_DATA story.csv posts

## SQL queries (Social media app)

### 1. Register a New User
INSERT INTO users (email, password, handle) VALUES ('newuser@gmail.com', '12345678', 'Adams');

### 2. Create a New Message
INSERT INTO posts (user_id, post_type, content, recipient_id, created_at, visible) 
VALUES (1, 'Message', 'Hello, how are you', 2, DATETIME('now'), TRUE);

### 3. Create a New Story by a Particular User
INSERT INTO posts (user_id, post_type, content, created_at, visible) 
VALUES (1, 'Story', 'I am happy today', DATETIME('now'), TRUE);

### 4. Show the 10 Most Recent Visible Messages and Stories, in Order of Recency
SELECT * FROM posts 
WHERE visible = TRUE 
ORDER BY created_at DESC 
LIMIT 10;

### 5. Show the 10 Most Recent Visible Messages Sent by a Particular User to a Particular User
SELECT * FROM posts 
WHERE visible = TRUE AND post_type = 'Message' AND user_id = 1 AND recipient_id = 2 
ORDER BY created_at DESC 
LIMIT 10;

### 6. Make All Stories That Are More Than 24 Hours Old Invisible
UPDATE posts 
SET visible = FALSE 
WHERE post_type = 'Story' AND (JULIANDAY('now') - JULIANDAY(created_at)) * 24 > 24;

### 7. Show All Invisible Messages and Stories, in Order of Recency
SELECT * FROM posts 
WHERE visible = FALSE 
ORDER BY created_at DESC;

### 8. Show the Number of Posts by Each User
SELECT user_id, COUNT(*) AS num_posts 
FROM posts 
GROUP BY user_id;

### 9. Show the Post Text and Email Address of All Posts and the User Who Made Them Within the Last 24 Hours
SELECT p.content, u.email 
FROM posts p 
JOIN users u ON p.user_id = u.user_id 
WHERE (JULIANDAY('now') - JULIANDAY(p.created_at)) * 24 <= 24;

### 10. Show the Email Addresses of All Users Who Have Not Posted Anything Yet
SELECT email 
FROM users 
WHERE user_id NOT IN (SELECT DISTINCT user_id FROM posts);
