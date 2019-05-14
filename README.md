<h2>Welcome</h2>

Welcome to Magento 2 installation! This repo is for installation of Magento 2 on DigitalOcean running Ubuntu 18.04. For more information about Magento 2, take a look at the original repo here. [https://github.com/magento/magento2/](https://github.com/magento/magento2/) magento2- Retested on Version 2.2.6 and still the LAMP droplet on 18.04.

# Basic requirements
* [Magento system requirements](https://devdocs.magento.com/guides/v2.2/install-gde/system-requirements.html), you can check this if you want, but you will get all the needed components running in your droplet if you follow this guide.
* [DigitalOcean Account](https://m.do.co/t/b298d6966c0c) - Click here for $10 referral credit, you will need a DigitalOcean account with a new one-click LAMP droplet running on Ubuntu 18.04, I have included my referral link, you can get 10 bucks which is enough for 2 months of the cheapest droplet, or 1 month of the 2GB-RAM droplet which is recommended for running Magento smoothly.
* [Magento prereq](https://devdocs.magento.com/guides/v2.2/install-gde/prereq/prereq-overview.html), if LAMP is not installed, make sure magento prereq is installed.
* [Magento Account](https://magento.com/tech-resources/download), you can use this repo to clone the code for installation, but it might be easier if you setup an account at Magento.com to download other stable versions, some with sample data, for your installation.
* [SFTP client](https://filezilla-project.org/), if you decide to download the zip files above, you will need SFTP client to upload it to your droplet.
* Domain name, for setting up SSL using Let's Encrypt. You might be able to use some DDNS services with it as well.

## Step 1 - Get your Ubuntu 16.04 Ubuntu Droplet
Get your droplet ready, log in with SSH, note down the passwords and the mysql root password from the fresh install.

## Step 2 - Set locale and run apt update/upgrade
```
pico /etc/default/locale
```
Add following lines to locale file:
```
LANG="en_US.UTF-8"
LANGUAGE="en_US:en"
LC_ALL="en_US.UTF-8"
```
Then run:
```
sudo apt-get update
sudo apt-get upgrade
```
## Step 3 install Apache
[Apache 2.4](https://devdocs.magento.com/guides/v2.2/install-gde/prereq/apache.html)
If you must install both Apache and PHP, install Apache first.
```
apt-get -y install apache2
```
check version
```
apache2 -v
```
Setup Magento site
```
sudo nano /etc/apache2/sites-available/magento.conf
```
Add the following lines:
```
<VirtualHost *:80>
    DocumentRoot /var/www/html
    <Directory /var/www/html/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
    </Directory>
</VirtualHost>
```
And run:
```
sudo a2ensite magento.conf
sudo a2dissite 000-default.conf
sudo a2enmod rewrite
sudo service apache2 restart
```
apache2 now is start.

## Step 4 install php
[php 7.0](https://devdocs.magento.com/guides/v2.2/install-gde/prereq/php-ubuntu.html)
```
sudo apt-get -y update
sudo add-apt-repository ppa:ondrej/php
sudo apt-get -y update
```
setup php
```
sudo nano /etc/php/7.0/apache2/php.ini
```
edit this line to increase memory limit
```
memory_limit = 512M
```
then run these:
```
sudo apt-get install -y php7.0 libapache2-mod-php7.0 php7.0-common php7.0-gd php7.0-mysql php7.0-mcrypt php7.0-curl php7.0-intl php7.0-xsl php7.0-mbstring php7.0-zip php7.0-bcmath php7.0-iconv php7.0-soap
sudo phpenmod mcrypt
```
Verify that PHP 7.0 is installed properly:
```
php -v
```
Verify that all required PHP extensions were installed:
```
php -me
```
If PHP is installed, continue with the next prerequisite, MySQL.

## Step 5 install mysql
[mysql 5.7.x](https://devdocs.magento.com/guides/v2.2/install-gde/prereq/mysql.html)
install
```
sudo apt install -y mysql-server mysql-client
```
set password
```
sudo mysql_secure_installation
```
check version
```
mysql --version
```
Start MySQL
```
mysql -u root -p
```
Run the following in MySQL to create database and user, choose a properly.
```
CREATE DATABASE magento;
CREATE USER magento_user@localhost IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON magento.* TO magento_user@localhost IDENTIFIED BY '<password>';
FLUSH PRIVILEGES;
exit
```

## Step 6 Install the Magento files
Download the Magento file archive to your home directory manually from Magento.com In my case, I downloaded Magento-CE-2.2.6_sample_data-2018-09-07-02-38-51.tar.gz.
```
mv Magento-CE-2.2.6_sample_data-2018-09-07-02-38-51.tar.gz /var/www/html/
cd /var/www/html/
tar xzvf Magento-CE-2.2.6_sample_data-2018-09-07-02-38-51.tar.gz
rm Magento-CE-2.2.6_sample_data-2018-09-07-02-38-51.tar.gz
sudo chown -R www-data:www-data /var/www/html/
```
tips：Download from magento.com no app/code directory, so download package from github to install. If unzip package to /var/www/html/ root directory, open ip address will have error "Autoload error
Vendor autoload is not found. Please run 'composer install' under application root directory.", which tell you install composer go to setp 7 

## Step 7 - optional: install composer
[build system include install composer](https://devdocs.magento.com/guides/v2.2/config-guide/deployment/pipeline/build-system.html)
First, check if Composer is already installed:

In a command prompt, enter any of the following commands:
```
composer --help
composer list --help
```
If command help displays, Composer is already installed.

If an error displays, use the following steps to install Composer.

To install Composer:

Change to or create an empty directory on your Magento server.

Enter the following commands:
```
curl -sS https://getcomposer.org/installer | php
```
shows message like this:
All settings correct for using Composer
Downloading...

Composer (version 1.8.4) successfully installed to: /root/composer.phar
Use it: php composer.phar
we need make it global use
```
mv /root/composer.phar /usr/bin/composer
```
check composer version
``` 
composer -v
```
tips：Do not run Composer as root/super user! See https://getcomposer.org/root for details

So we need create a user, be sure to replace username with the user that you want to create, we use "magento".
```
adduser username
passwd username
```
if check /home No directory, logging in with HOME=/
```
mkdir /home/magento
sudo usermod --shell /bin/bash --home /home/magento magento

```
Use the usermod command to add the user to the sudo group.
```
usermod -aG sudo magento
```
Use the su command to switch to the new user account.
```
su magento
```
then run following command under /var/www/html/
```
composer install
```
if fail, add permission
```
sudo chmod 777 -R html
```
check permission, then install again.
```
sudo ls -l /var/www
```
get message "Generating autoload files".

## tips: using unzip package may has error, 404 not found, please make sure all files in the directory.
how to solve
I have faced the same problem after wasting lot of time solved it just adding the .htaccess file in root folder.

## Step 8 - Setup Let's Encrypt
You need to have mapped your domain name to the IP address. Follow the setup screens below, I suggest not setting up for force all HTTPS traffic, you can do that inside Magento setup later.
```
cd /usr/local/sbin
sudo wget https://dl.eff.org/certbot-auto
sudo chmod a+x /usr/local/sbin/certbot-auto
certbot-auto --apache -d <domain name, without https:// part, for example, www.github.com>
```
You can check your certificate using the following URL. https://www.ssllabs.com/ssltest/analyze.html?d=<domain name, without https:// part, for example, www.github.com>&latest

## Step 8 - Setup Cron Jobs for indexes and auto Let's Encrypt renewal
```
sudo crontab -e
```
Add the following lines:
```
30 2 * * 1 /usr/local/sbin/certbot-auto renew >> /var/log/le-renew.log
* * * * * /usr/bin/php /var/www/html/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /var/log/magento.cron.log
* * * * * /usr/bin/php /var/www/html/update/cron.php >> /var/log/update.cron.log
* * * * * /usr/bin/php /var/www/html/bin/magento setup:cron:run >> /var/log/setup.cron.log
```
## Step 9 - Restart Apache
```
service apache2 restart
```
## Step 10 - Web setup of Magento - Well Done!
Visit your new domain for the first time to get started. You will need the username and password you setup in MySQL for the database user access. It should now be working. Well done!

## Other Useful Commands
To reindex Magento
```
php /var/www/html/bin/magento indexer:reindex
```
Check your php path
```
which php
```
## Other Installation Documents
[Magento DevBox](https://magento.com/tech-resources/download), the easiest way to get started with Magento.
[Installation guide](http://devdocs.magento.com/guides/v2.2/install-gde/bk-install-guide.html)


