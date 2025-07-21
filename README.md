# docker-html-php-mysql
<pre>
Question: 

You are tasked with creating a simple Dockerized PHP web application that provides a login interface. The system should meet the following requirements: 

Use a PHP-Apache container to serve an HTML sign-in form and a PHP backend (signin.php) that validates user credentials. 

Use a MySQL container as the backend database, with a database named signin_db and a table users containing email and password fields. 

Ensure communication between containers using a custom Docker network. 

Persist MySQL data using a Docker volume. 

Use a Dockerfile to install the required mysqli extension in the PHP container. 

Insert a test user (test@example.com / password123) into the database to verify login functionality. 

Task: 
 Describe the docker-compose.yml, Dockerfile, and necessary PHP/MySQL steps used to complete this task. 
</pre> 
--------------------------------------------------
### Step 1:
### Q1: How do you create a persistent storage volume for MySQL data? 

✅ Answer: docker volume create db-data 

### Q2: How do you create a custom Docker network so services can communicate? 

✅ Answer: docker network create app-network 
--------------
### Step 2: Create docker-compose.yml 

### Q3: What should your docker-compose.yml look like to define PHP and MySQL services with custom volume and network? 
<pre>
version: "3.8"

services:
  php:
    build: .
    container_name: php-app
    volumes:
      - .:/var/www/html
    ports:
      - "8080:80"
    networks:
      - app-network
    depends_on:
      - mysql

  mysql:
    image: mysql:8.0
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: signin_db
      MYSQL_USER: user
      MYSQL_PASSWORD: userpass
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - app-network

networks:
  app-network:

volumes:
  db-data:  
  
</pre>

### Step 3: Create Dockerfile for PHP with MySQLi support 

**Q4: What Dockerfile is used to add mysqli support in the PHP container?**
<pre>
  
FROM php:8.2-apache

RUN docker-php-ext-install mysqli
  
</pre>
### Step 4: Create HTML and PHP Files
**Q5: What does your index.html look like for the login form?**

✅ Answer:
  
```<!DOCTYPE html>
<html>
<head><title>Sign In</title></head>
<body>
  <h2>Sign In</h2>
  <form action="signin.php" method="POST">
    <label>Email:</label><br>
    <input type="email" name="email" required><br>
    <label>Password:</label><br>
    <input type="password" name="password" required><br><br>
    <input type="submit" value="Sign In">
  </form>
</body>
</html>
```
### Q6: What is the content of signin.php that handles the form submission?

**✅ Answer:**
<pre>
<?php
require 'db.php';

$email = $_POST['email'];
$password = $_POST['password'];

$stmt = $conn->prepare("SELECT * FROM users WHERE email=? AND password=?");
$stmt->bind_param("ss", $email, $password);
$stmt->execute();
$result = $stmt->get_result();

if ($result->num_rows > 0) {
    echo "Login Successful";
} else {
    echo "Invalid credentials";
}
?>
</pre>

### Q7: What is the content of db.php to connect to MySQL?

**✅ Answer:**
<pre>
<?php
$host = 'mysql-db';
$db = 'signin_db';
$user = 'user';
$pass = 'userpass';

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
?>
</pre>

### Step 5: Start and Test the App
**Q8: How do you start the application?**


**✅ Answer:**
<pre>
docker-compose up --build
</pre>

### Step 6: Create Users Table and Insert Data
**Q9: How do you log in to MySQL inside the container?**

**✅ Answer:**
<pre>
docker exec -it mysql-db mysql -u user -p
</pre>
**Password: userpass**

### Q10: What SQL commands do you run to create the users table and insert a user?

**✅ Answer:**
<pre>
USE signin_db;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL
);

INSERT INTO users (email, password) VALUES ('test@example.com', 'password123');
</pre>

Now you can log in using:

Email: test@example.com

Password: password123

 
