# (STEP 4) PROJECT 2 - LEMP STACK IMPLEMENTATION.


## STEP 1 - INSTALLING THE NGINX WEB SERVER.

Since this is our first time using apt for this session, start off by updating the server’s package index. 


`sudo apt update`

`sudo apt install nginx`


![Server Package Index Update](./Images/Server%20Package%20Index%20Update.PNG)


 Verify that nginx was successfully installed and is running as a service in Ubuntu, run:


`sudo systemctl status nginx`


![Nginx Successfully Installed](./Images/Nginx%20Successfully%20Installed.PNG)


Open TCP port 80 which is default port that web brousers use to access web pages in the Internet.

As we know, we have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80:

First, check how we can access it locally in the Ubuntu shell, run:


`curl http://localhost:80`

or

`curl http://127.0.0.1:80`


![Server Check Local](./Images/Server%20Check%20Local.PNG)


Test how the Nginx server can respond to requests from the Internet.
- Open a web browser
- Get the public address or the DNS from the instance.
- Paste the public address (3.20.225.224) in the browser and look for the output.


![Nginx Welcome Page](./Images/Nginx%20Welcome%20Page.PNG)


## STEP 2 - INSTALLING MYSQL


Now that we have a web server up and running, we need to install a Database Management System (DBMS) to be able to store and manage data for the site in a relational database.


Use ‘apt’ to acquire and install this software.

`sudo apt install mysql-server`


![MYSQL Install](./Images/MYSQL%20Install.PNG)


When the installation is finished, log in to the MySQL console by typing:

`sudo mysql`


![MYSQL Login](./Images/MYSQL%20Login.PNG)


Run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Before running the script you will set a password for the root user, using mysql_native_password as default authentication method. We’re defining this user’s password as PassWord.1.

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`


![Root Password Set](./Images/Root%20Password%20Set.PNG)


Exit the MySQL shell with:

`exit`


![Exit MYSQL](./Images/Exit%20MYSQL.PNG)


Start the interactive script by running:

`sudo mysql_secure_installation`


Enter PassWord.1 as the password.


Answer Y for yes, or anything else to continue without enabling. In our case enter Yes.


Enter 1 which is equal to Medium . This will be our level of password validation:


When prompted to change password for root, select No.


For the rest of the questions, press Y and hit the ENTER key at each prompt. 


![Validate Password Plugin1](./Images/Validate%20Password%20Plugin1.PNG)


![Validate Password Plugin](./Images/Validate%20Password%20Plugin2.PNG)


Test if you’re able to log in to the MySQL console by typing:

`sudo mysql -p`

You will be prompted to input the password . Use PassWord.1 as the root password. The -p flag in this command, which will prompt you for the password used after changing the root user password.


![MYQSL Login Test](./Images/MYSQL%20Login%20Test.PNG)


Exit the MySQL console by the command below.

`exit`


![Exit MYSQL2](./Images/Exit%20MYSQL2.PNG)



## STEP 3 - INSTALLING PHP


*Nginx is installed to serve your content and MySQL is installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.*



*While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration.* 

*We need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.* 


Core PHP packages will automatically be installed as dependencies.


To install these 2 packages at once, run:


`sudo apt install php-fpm php-mysql`


When prompted, type Y and press ENTER to confirm installation.


![PHP Components Installed](./Images/PHP%20Components%20Installed.PNG)



## STEP 4 - CONFIGURING NGINX TO USE PHP PROCESSOR.


*When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use projectLEMP as an example domain name.*



*On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.*


Create the root web directory for your_domain as follows:


![Create Root Web Directory](./Images/Create%20Root%20Web%20Directory.PNG)


Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:


`sudo chown -R $USER:$USER /var/www/projectLEMP`


![Assign Directory Ownership](./Images/Assign%20Directory%20Ownership.PNG)


Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:


`sudo nano /etc/nginx/sites-available/projectLEMP`


This will create a new blank file. Paste in the following bare-bones configuration:


#/etc/nginx/sites-available/projectLEMP
```
server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```


When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.



![Configuration File Nano](./Images/Configuration%20File%20Nano.PNG)


Activate the configuration by linking to the config file from Nginx’s sites-enabled directory:


`sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`


![Configuration Activation](./Images/Configuration%20Activation.PNG)


This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:


`sudo nginx -t`


![Configuration Test](./Images/Configuration%20Test.PNG)


Disable default Nginx host that is currently configured to listen on port 80, for this run:


`sudo unlink /etc/nginx/sites-enabled/default`


![Disable Default Nginx Host](./Images/Disable%20Default%20Nginx%20Host.PNG)


Reload Nginx to apply the changes:


`sudo systemctl reload nginx`


![Reload Nginx](./Images/Reload%20Nginx.PNG)


*Your new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:*


`sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html`


![index.html file](./Images/index.html%20file.PNG)


Now go to your browser and try to open your website URL using IP address: You can get the Public IP address from the instance.


`http://18.117.247.58:80`


![Website Output](./Images/Website%20Output.PNG)


*You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.*

*Your LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.*



## STEP 5 - TESTING PHP WITH NGINX.



*You can test the LEMP stack which is fuilly functional to validate that Nginx can correctly hand .php files off to your PHP processor.*

*You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:*


`sudo nano /var/www/projectLEMP/info.php`


Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:

```
<?php
phpinfo();
```

Save by CTRL + X and then Y and Enter to confirm.


![Valid PHP Code Information](./Images/Valid%20PHP%20Code%20Information.PNG)


You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:


http://18.117.247.58/info.php


You will see a web page containing detailed information about your server:


![Web page PHP](./Images/Web%20page%20PHP.PNG)


*After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file:*


`sudo rm /var/www/projectLEMP/info.php`


![Remove File](./Images/Remove%20File.PNG)


Refresh the web browser and you will see the output below.


![404 Not Found](./Images/404%20Not%20Found.PNG)



## STEP 6 - RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED).


*In this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.*


*At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.*


*We will create a database named **example_database** and a user named **example_user**, but you can replace these names with different values.*

First, connect to the MySQL console using the root account:


`sudo mysql -p`

![MYSQL Login2](./Images/MYSQL%20Login2.PNG)


To create a new database, run the following command from your MySQL console:


`CREATE DATABASE 
`example_database`;`


![Created Database](./Images/Created%20Database.PNG)


Create a new user and grant it full privileges on the database you have just created.

The following command creates a new user named **example_user**, using mysql_native_password as default authentication method. We’re defining this user’s password as **PassWord.2**, but you should replace this value with a secure password of your own choosing.


`CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.2';`


![New user](./Images/New%20User.PNG)


Give this user permission over the example_database database:


`GRANT ALL ON example_database.* TO 'example_user'@'%';`


![Permission Granted](./Images/Permission%20Granted.PNG)


*This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.*

Now exit the MySQL shell with:


`exit`

![Exit MYSQL3](./Images/Exit%20MYSQL2.PNG)


You can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:


`mysql -u example_user -p`


![MYSQL Login3](./Images/MYSQL%20Login3.PNG)


*Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user. After logging in to the MySQL console, confirm that you have access to the example_database database: I used **PassWord.2** as the password*


`SHOW DATABASES;`


![Show Database](./Images/Show%20Database.PNG)


Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:


`CREATE TABLE example_database.todo_list (

item_id INT AUTO_INCREMENT,

content VARCHAR(255),

PRIMARY KEY(item_id)

);`


![Test Table Created](./Images/Test%20Table%20Created.PNG)


Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:


`INSERT INTO example_database.todo_list (content) VALUES ("My first important item");`


`INSERT INTO example_database.todo_list (content) VALUES ("I am very tired and almost done with Project 2");`


`INSERT INTO example_database.todo_list (content) VALUES ("Doing this one step at a time to be a DevOps Engineer");`


![Insert Rows](./Images/Insert%20Rows.PNG)


To confirm that the data was successfully saved to your table, run:


`SELECT * FROM example_database.todo_list;`


Below is the output.


![Rows Output](./Images/Rows%20Output.PNG)


After confirming that you have valid data in your test table, you can exit the MySQL console:


`exit`


![Exit MYSQL4](./Images/Exit%20MYSQL4.PNG)


*Now you can create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. We’ll use vi for that:*


`nano /var/www/projectLEMP/todo_list.php`


*The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.*


Copy this content into your todo_list.php script:

```
<?php
$user = "example_user";
$password = "PassWord.2";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```

Save and close the file when you are done editing. To save use CTRL + X and then Y and enter to confirm.


![PHP Script MYSQL Connect](./Images/PHP%20Script%20MYSQL%20Connect.PNG)


You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:


http://18.117.247.58/todo_list.php


![Todo List Web Output](./Images/Todo%20List%20Web%20Output.PNG)


**The PHP environment is ready to connect and interact with the MySQL server.**
































