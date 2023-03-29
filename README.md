# Nextcloud

How to Install and Configure Nextcloud Hub 21
March 29, 2023 by Still-not-found

Table of Contents:

Install Apache & MySQL

Prepare MySQL database

Install PHP for Nextcloud

Download Nextcloud Hub 3 or any version you need

Configure Apache for Nextcloud

Nextcloud Configuration

Fixing Installation Issues

Update PHP Configuration

Set up Cronjob for Nextcloud

No default phone region set

No memory cache has been configured

Fix php-imagick warning

Install ONLYOFFICE
Move Nextcloud data directory

Further Nextcloud Configuration

Set up SMTP

This tutorial explains how to install and configure Nextcloud Hub version 25 on Ubuntu Server 20.04 LTS.
I show how to configure Apache and how to set up a MySQL user and database for Nextcloud Hub.
I also walk you through common security & setup warning issues that often come up when installing Nextcloud.
Finally, I explain how to properly configure PHP, set up a cronjob, ONLYOFFICE, and SMTP.

Install Apache & MySQL
Most likely, you already have Apache2 and MySQL installed on your server. If not, you can quickly install them with

sudo apt update
sudo apt install apache2
sudo apt install mysql-server
Before setting up a new user and database, you should secure your MySQL server (again, you can skip this step if you already have used MySQL before)

sudo mysql_secure_installation
VALIDATE PASSWORD COMPONENT: n
New root passwort: <YOUR MYSQL PASSWORD>
Re-enter new password: <YOUR MYSQL PASSWORD>
Remove anonymous user: y
Disallow root login remotely: y
Remove test database and access to it: y
Reload privileges tables now: y

Prepare MySQL database
In a next step, you want to create a new user and database on your MySQL server:

sudo mysql -u root -p
<ENTER YOUR MYSQL PASSWORD>
mysql> create database nextcloud;
mysql> create user 'nextcloud'@'localhost' identified by 'PASSWORD';
mysql> grant all privileges on nextcloud.* to 'nextcloud'@'localhost';
mysql> flush privileges;
mysql> quit
Install PHP for Nextcloud
For Nextcloud to work, you will need a new version of PHP as well as a few PHP libraries. Likely, you have already installed a version of PHP but its also no problem to have multiple versions installed on your server. We will talk about how to enable the latest installed PHP version in Apache in the section Configure Apache for Nextcloud below.

sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt upgrade
Install PHP with all required libraries:

sudo apt install php8.0
sudo apt install php8.0-gd php8.0-mysql php8.0-curl php8.0-mbstring php8.0-apcu
sudo apt install php8.0-intl php8.0-gmp php8.0-bcmath php8.0-xml
sudo apt install libapache2-mod-php8.0 php8.0-zip php-imagick redis-server php-redis
As of writing this article, Nextcloud does not work with PHP8.1 so make sure to use PHP8.0 instead!

Download Nextcloud Hub 21
Go to https://nextcloud.com
Click on “Get Nextcloud”
Click on “Server packages”
Right-click on “Download Nextcloud”
Click on “Copy link address”
Then, download the latest version of Nextcloud directly onto your home server:

wget https://download.nextcloud.com/server/releases/nextcloud-23.0.0.zip
Next, unzip Nextcloud directly into the www directory of Apache (first run sudo apt install unzip if you don’t have unzip installed yet):

sudo unzip nextcloud-23.0.0.zip -d /var/www
Switch directories and change ownership of the Nextcloud folder to Apache, which is the “www-data” user:

cd /var/www
sudo chown -R www-data:www-data nextcloud/

Configure Apache for Nextcloud
First, you need to enable a few Apache modifications for Nextcloud to properly run

sudo a2enmod headers env dir mime rewrite
sudo service apache2 restart
Add a VirtualHost entry in the default configuration file of Apache. If you already have a domain name, you can additionally specify it under the DocumentRoot directive. Don’t worry if you don’t have your own domain – you can just as easily use Nextcloud only locally without the need of a domain name.

/etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>

    ServerName cloud.yourdomain.com
    DocumentRoot /var/www/nextcloud

    <Directory /var/www/nextcloud/>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews

        <IfModule mod_dav.c>
            Dav off
        </IfModule>

        RewriteEngine On
        RewriteRule ^/\.well-known/carddav https://%{SERVER_NAME}/remote.php/dav/ [R=301,L]
        RewriteRule ^/\.well-known/caldav https://%{SERVER_NAME}/remote.php/dav/ [R=301,L]
        RewriteRule ^/\.well-known/host-meta https://%{SERVER_NAME}/public.php?service=host-meta [QSA,L]
        RewriteRule ^/\.well-known/host-meta\.json https://%{SERVER_NAME}/public.php?service=host-meta-json [QSA,L]
        RewriteRule ^/\.well-known/webfinger https://%{SERVER_NAME}/public.php?service=webfinger [QSA,L]

    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
Finally, restart Apache

sudo service apache2 restart
Optionally, you can also secure your Nextcloud installation using SSL. You can find a tutorial on how to enable SSL for your Nextcloud installation here.


