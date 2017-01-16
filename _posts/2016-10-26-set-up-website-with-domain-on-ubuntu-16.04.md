---
layout: post
title: "Setup Website with Domain Name Based on WordPress on Ubuntu 16.04"
date: 2016-10-26 16:27
comments: true
categories: Linux
---


I have the idea to build a website for many years. There are many excuses why I didn't, one important is there are many free alternatives, like github project blog. But now, I want to build one for myself after I bought the advanced Acer G3-710. 


I bought a domain name: **shinyapps.me**. Quite cool, right? There is website called, **shinyapps.io**, if you know *shiny*, you know how popular it is. I want to build a website which can hold all shiny apps that I created.  It’s not so difficult than expected: I spent two days, but most of the time spent is to choose which solution (framework). Finally, I decided to use the framework that most websites using: **Apache** + **MySQL** + **PHP**. **Nodejs** + **MongoDB** could be an alternative choice, but compared with PHP, there is not so many help documents. 


Now, let’s start! 

## Install LAMP (Linux, Apache, MySQL and PHP)


~~~~ 
sudo apt-get -y install tasksel
sudo tasksel install lamp-server
~~~~ 

This will install all software needed for building the website. 


- **Test Apache**. Once fished, Type the IP address in a web browser, e.g. http://188.166.116.72/. If you can see the following page, then the installation succeeded.


![]( /images/shinyapps/testApache.png)

The above page is actually the file: /var/www/html/index.html. Now, let’s do some change.Delete the above file, and create a new one with same name. Write **<p> HTML test success! </p>** into the new created file, then refresh the browser. You should can see that the content have changed to **HTML test success!**. 

- **Test PHP**. Add a php file in **/var/www/html**, say *test.php*, which content as **<?php echo(exec("whoami")); ?>**
Now go to web browser and type: *IP/test.php*. We should can get “www-data”

- **Setup MySQL**. 

~~~~ 
sudo mysql -u root -p
mysql> CREATE DATABASE myDataBase;
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES ON myDataBase.* TO 'UserName'@'localhost' IDENTIFIED BY 'Password';

~~~~ 

Where **myDataBase** is the Database Name, **UserName** is the username, and ***Password* is the password. Above information will required when you first run your web site (wordpress configure).


## Install WordPress

~~~~ 
cd /var/www
sudo wget http://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
~~~~ 

We need to modify file **/etc/apache2/sites-available/000-default.conf **
to change the default directory to WordPress: **/var/www/wordpress**.

And change the access permission, so WordPress can access the following directory:

~~~~ 
sudo chown -R www-data: /var/www/wordpress
~~~~ 

Now restart the apache: 


~~~~ 
sudo /etc/init.d/apache2 restart
~~~~


You should be able to see this page: 

![]( /images/shinyapps/wordpress1.png)

![]( /images/shinyapps/wordpress2.png)

![]( /images/shinyapps/wordpress3.png)

Follow the instruction to configure your website. When you feel you have done enough, go back to the IP address of your server.

You can log onto your WordPress to customize your website further, like install a theme call "**Wilse**". 


## Setup domain name

- **Godaddy side** 

I bought the domain name from **godaddy.com**, so log into your account, and navigate to your domain name, change like the following figure, replace with your IP address

![]( /images/shinyapps/godaddy1.png)
![]( /images/shinyapps/godaddy2.png)

- **WordPress side**

Log into your wordpress and set the domain name and wordpress name to the domain name, like **http://shinyapps.me**. Note make sure it's your domain name, not something others!

![]( /images/shinyapps/wordpress4.png)







 



