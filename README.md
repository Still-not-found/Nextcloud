
# Nextcloud Installation In Debian/Ubuntu Servers

How to Install and Configure Nextcloud Hub 25 or later versions.

March 29, 2023 by Still-not-found


## Description:
This tutorial explains how to install and configure Nextcloud Hub version 25 on Ubuntu Server 20.04 LTS. I show how to configure Apache and how to set up a MySQL user and database for Nextcloud Hub. I also walk you through common security & setup warning issues that often come up when installing Nextcloud. Finally, I explain how to properly configure PHP, set up a cronjob, ONLYOFFICE, and SMTP.

## Install Apache & MySQL
Most likely, you already have Apache2 and MySQL installed on your server. If not, you can quickly install them with









```bash
  sudo apt update
  sudo apt install apache2
  sudo apt install mysql-server

```
 Before setting up a new user and database, you should secure your MySQL server (again, you can skip this step if you already have used MySQL before)

 ```bash
sudo mysql_secure_installation
VALIDATE PASSWORD COMPONENT: n
New root passwort: <YOUR MYSQL PASSWORD>
Re-enter new password: <YOUR MYSQL PASSWORD>
Remove anonymous user: y
Disallow root login remotely: y
Remove test database and access to it: y
Reload privileges tables now: y
```
## Prepare MySQL database

In a next step, you want to create a new user and database on your MySQL server:

```bash
sudo mysql -u root -p
<ENTER YOUR MYSQL PASSWORD>
mysql> create database nextcloud;
mysql> create user 'nextcloud'@'localhost' identified by 'PASSWORD';
mysql> grant all privileges on nextcloud.* to 'nextcloud'@'localhost';
mysql> flush privileges;
mysql> quit
```

## Install PHP for Nextcloud

For Nextcloud to work, you will need a new version of PHP as well as a few PHP libraries. Likely, you have already installed a version of PHP but its also no problem to have multiple versions installed on your server. We will talk about how to enable the latest installed PHP version in Apache in the section Configure Apache for Nextcloud below.

 ```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt upgrade
```
Install PHP with all required libraries:

```bash
sudo apt install php8.0
sudo apt install php8.0-gd php8.0-mysql php8.0-curl php8.0-mbstring php8.0-apcu
sudo apt install php8.0-intl php8.0-gmp php8.0-bcmath php8.0-xml
sudo apt install libapache2-mod-php8.0 php8.0-zip php-imagick redis-server php-redis

```

## Download Nextcloud Hub (25 or above)

* Go to https://nextcloud.com
* Click on “Get Nextcloud”
* Click on “Server packages”
* Right-click on “Download Nextcloud”
* Click on “Copy link address”

Then, download the latest version of Nextcloud directly onto your home server:

```bash
wget https://download.nextcloud.com/server/releases/nextcloud-25.0.1.zip

```
Next, unzip Nextcloud directly into the www directory of Apache (first run sudo apt install unzip if you don’t have unzip installed yet):

```bash
sudo unzip nextcloud-25.0.1.zip -d /var/www/

```
Switch directories and change ownership of the Nextcloud folder to Apache, which is the “www-data” user:
```bash
cd /var/www
sudo chown -R www-data:www-data nextcloud/

```
 
 ## Configure Apache for Nextcloud

 First, you need to enable a few Apache modifications for Nextcloud to properly run
 ```bash
sudo a2enmod headers env dir mime rewrite
sudo service apache2 restart

```
Add a VirtualHost entry in the default configuration file of Apache. If you already have a domain name, you can additionally specify it under the DocumentRoot directive. Don’t worry if you don’t have your own domain – you can just as easily use Nextcloud only locally without the need of a domain name.

### /etc/apache2/sites-available/000-default.conf

```bash
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
```
Finally, restart Apache

```bash
sudo service apache2 restart
```
In my case, my Nextcloud server is located at 192.168.1.xx, hence I need to update the Nextcloud configuration file in order to be able to use that local IP address to browse to my Nextcloud instance. Also, this is where you can specify your domain if you have one:

### /var/www/nextcloud/config/config.php
```bash
'trusted_domains' => 
array(
  0 => 192.168.1.xx:80
  1 => http://yourdomain.com
),
```
You should now be able to access your Nextcloud instance by browsing to http://YOUR-SERVER-LOCAL-IP

* add your username and password
* The path will automatically deducted there /var/www/nextcloud/data
* add your database user and password
* And click the install button. 

### Fixing Installation Issues
Click on settings and then on “Overview” to see on-going issues with your Nextcloud installation. It is completely normal for a new installation to produce a few errors, so here is how you can fix the most common issues

### Update PHP Configuration
Any fresh Apache & PHP installation will trigger the following warning:
        * The PHP memory limit is below the recommended value of 512MB.
This can easily be fixed by increasing the respective value in the php.ini file:
```bash
memory_limit = 512M
```

### Set up Cronjob for Nextcloud
For Nextcloud to run smoothly, you will want to set up a cronjob. This is a task that is executed automatically in the background. Modify your Apache cronjob:
```bash
sudo crontab -u www-data -e
```
If asked, press “1” to use the nano editor (which is super easy to use) and add the following line to your crontab file:
```bash
*/5  *  *  *  * php -f /var/www/nextcloud/cron.php
```
This will execute the Nextcloud cronjob every 5 Minutes. You can use this website to better understand how to modify the execution period:

### No memory cache has been configured
It is highly recommended to set up some form of memory caching in order to improve your Nextcloud’s performance. I recommend using Redis for this purpose, which we have already previously installed during the PHP installation process.
```bash
nextcloud@nextcloud:~$ ps ax | grep redis
2488834 ?        Ssl    1:24 /usr/bin/redis-server 127.0.0.1:6379
```
Add the Apache user (www-data) to the redis group and restart the web server
```bash
sudo usermod -a -G redis www-data
sudo service apache2 restart
```
Finally, simply add the following lines to your Nextcloud configuration

#### go to /var/www/nextcloud/config/config.php
```bash
'filelocking.enabled' => true,
  'memcache.local' => '\\OC\\Memcache\\APCu',
  'memcache.locking' => '\\OC\\Memcache\\Redis',
  'redis' =>
  array (
    'host' => '127.0.0.1',
    'port' => 6379,
  ),
```
This will enable file locking to avoid corrupting files and specifies Redis as local caching server. We need to explicitly state that we want to use the Redis server running on our localhost (127.0.0.1) and on the default port (6379).

### Fix php-imagick warning
On a fresh installation of Ubuntu 20.04 you will likely get the following warning message:
       * Module php-imagick in this instance has no SVG support. For better compatibility it is recommended to install it.
Then you will have to first uninstall ImageMagick and all its dependencies using:
```bash
sudo apt remove imagemagick-6-common php-imagick
sudo apt autoremove
```
and then reinstall imagemagick;
```bash
sudo apt install imagemagick php-imagick
```
### Install ONLYOFFICE

    * Click on your user image
    * Click on Apps
    * Select Office & Text from the left menu
    * Download and enable the Community Document Server
    * Download and enable ONLYOFFICE
    * Click on your user image and go to settings
    * Make sure to update your Document Editing Service address

### Move Nextcloud data directory

You can move the Nextcloud home directory to any drive of your liking if you want to save files that your Nextcloud serves on another drive than your OS drive (which you probably want since you likely won’t have terrabytes of space on your OS drive):

First, make sure that the External storage support is enabled:
```bash
sudo cp -a /var/www/nextcloud/data/. /mnt/cloud/data/
```
Next, update the Nextcloud configuration to reflect the new data directory:
#### go to : /var/www/nextcloud/config/config.php
```bash
'datadirectory' => '/mnt/cloud/data',
```
### Further Nextcloud Configuration
If Nextcloud is not installed in a subfolder, update the .htaccess file
```bash
sudo -u www-data php /var/www/nextcloud/occ maintenance:update:htaccess 
```
### Set up SMTP
Attention: In order to set up SMTP and enable email notifications from Nextcloud you will need to have a valid SMTP server.

First, you will need to actually set an Email address for your Nextcloud user:
  * Next, enter your SMTP details under “Basic settings”:

Finally, if you are running a firewall (which you should!), you will need to allow two ports for SMTP to work:
```bash
sudo ufw allow 465
```
 Congratulations, you should now have a working version of Nextcloud Hub version 25 up and running!






















 
