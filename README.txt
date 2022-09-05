                                                                      Magento with Docker-compose

#Setting Up Magento 2 Using Docker-Compose

#create directory Magento

$mkdir magento 

$cd magento

magento$ 

#To begin with create a directory on your Ubuntu 20.04 server for this project. Our directory architecture will be something like:

docker-compose.yaml           ![file]cd /magento/docker-compose.yaml   

   web-server                 ![directory]cd /magento/web-server
     Dockerfile               ![file]cd /magento/web-server/Dockerfile
     supervisord.conf         ![file]cd /magento/web-server/supervisord.conf 

   database_server            ![directory]cd magento/database_server
      Dockerfile              ![file]cd /magento/database_server/Dockerfile
      mysql.sh                ![file]cd /magento/database_server/mysql.sh
      supervisord.conf        ![file]cd /magento/database_server/supervisord.conf 
   
   magento2                   ![directory] in this direcory you have download your magento version you want
      install 
         sudo wget https://codeload.github.com/magento/magento2/zip/refs/tags/2.4.3
         sudo apt install unzip
         unzip 2.4.3
         sudo mv magento2-2.4.3 /home/ubuntu/magento/magento2
         sudo rm 2.4.3
         sudo rm -rf magento2-2.4.3


#Create a docker-compose.yaml file defining services that needed for application run

magento$ nano docker-compose.yaml

#add this instruction

------------------------------------------------------------------------------------------------------------------------------------------------------
version: '3'
services:
  web_server:
    build:
      context: ./web_server/
    container_name: apache2
    volumes:
      - ./magento2:/var/www/html
      - ./web_server/supervisord.conf:/etc/supervisor/conf.d/supervisord.conf
    network_mode: host

  database_server:
    build:
      context: ./database_server/
      args:
        - mysql_password=password
        - mysql_database=magentodb
    container_name: mysql
    volumes:
      - ./database_server/supervisord.conf:/etc/supervisor/conf.d/supervisord.conf
      - ./database_server/mysql.sh:/etc/mysql.sh
    network_mode: host
----------------------------------------------------------------------------------------------------------------------------------------------------------

#create directory web-server inside magento directory

cd magento
 
magento$ mkdir web_server

cd magento/web_server

create Dockerfile inside web_server

web_server$ nano Dockerfile

#ADD THIS LINE ON DOCKERFILE


FROM ubuntu:20.04
FROM ubuntu:latest
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
    && apt-get -y install apache2 nano mysql-client \
    && a2enmod rewrite \
    && a2enmod headers \
    && export LANG=en_US.UTF-8 \
    && apt-get update \
    && apt-get install -y software-properties-common curl \
    && apt-get install -y language-pack-en-base \
    && LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php \
    && apt-get update \
    && apt-get -y install php7.4 php7.4-curl php7.4-intl php7.4-gd php7.4-dom php7.4-mcrypt php7.4-iconv php7.4-xsl php7.4-mbstring php7.4-ctype   php7.4-zip php7.4-pdo php7.4-xml php7.4-bz2 php7.4-calendar php7.4-exif php7.4-fileinfo php7.4-json php7.4-mysqli php7.4-mysql php7.4-posix php7.4-tokenizer php7.4-xmlwriter php7.4-xmlreader php7.4-phar php7.4-soap php7.4-mysql php7.4-fpm php7.4-bcmath libapache2-mod-php7.4 \
    && sed -i -e"s/^memory_limit\s*=\s*128M/memory_limit = 4G/" /etc/php/7.4/apache2/php.ini \
    && rm /var/www/html/* \
    && sed -i "s/None/all/g" /etc/apache2/apache2.conf \
    && chmod 777 /usr/local/bin/ \
    && apt-get install curl \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=1.10.20 \
    && apt-get install composer -y \
    && apt-get install -y supervisor \
    && mkdir -p /var/log/supervisor
RUN apt-get update && apt-get install wget build-essential gcc make -y
RUN apt-get install default-jdk -y
RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.0-amd64.deb
RUN dpkg -i elasticsearch-7.10.0-amd64.deb
env APACHE_RUN_USER    www-data
env APACHE_RUN_GROUP   www-data
env APACHE_PID_FILE    /var/run/apache2.pid
env APACHE_RUN_DIR     /var/run/apache2
env APACHE_LOCK_DIR    /var/lock/apache2
env APACHE_LOG_DIR     /var/log/apache2
env LANG               C
WORKDIR /var/www/html
CMD ["/usr/bin/supervisord"]

------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#create supervisord.conf file inside web_server

web_server$ nano  supervisord.conf

#add this line  supervisord.conf

[supervisord]
nodaemon=true

[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"

[program:user_permission]
command=/bin/bash -c "chown -R www-data: /var/www/"
command=/bin/bash -c "composer install: /var/www/html/"

------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#create directory database_server inside magento

cd magento/database_server


#create Dockerfile inside database_server

nano Dockerfile

#add this line on dockerfile

FROM ubuntu:20.04
ARG mysql_password
ARG mysql_database
env MYSQL_ROOT_PASSWORD ${mysql_password}
env MYSQL_DATABASE ${mysql_database}

RUN apt-get update \
&& echo "mysql-server-8.0 mysql-server/root_password password ${mysql_password}" | debconf-set-selections \
&& echo "mysql-server-8.0 mysql-server/root_password_again password ${mysql_password}" | debconf-set-selections \
&& DEBIAN_FRONTEND=noninteractive apt-get -y install mysql-server-8.0 && \
    mkdir -p /var/lib/mysql && \
    mkdir -p /var/run/mysqld && \
    mkdir -p /var/log/mysql && \
    touch /var/run/mysqld/mysqld.sock && \
    touch /var/run/mysqld/mysqld.pid && \
    chown -R mysql:mysql /var/lib/mysql && \
    chown -R mysql:mysql /var/run/mysqld && \
    chown -R mysql:mysql /var/log/mysql &&\
    chmod -R 777 /var/run/mysqld/ \
    && sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/mysql.conf.d/mysqld.cnf \
    && apt-get install -y supervisor nano \
    && mkdir -p /var/log/supervisor
CMD ["/usr/bin/supervisord"]

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

cd magento/database_server

nano mysql.sh

#Add this line on mysql.sh    

                                                                                                                                                                                                 #!/bin/bash
set -u
sleep 4
database_connectivity_check=no
var=1
while [ "$database_connectivity_check" != "mysql" ]; do
/etc/init.d/mysql start
sleep 2
database_connectivity_check=`mysqlshow --user=root --password=$MYSQL_ROOT_PASSWORD | grep -o mysql`
if [ $var -ge 4 ]; then
exit 1
fi
var=$((var+1))
done
 
 
database_availability_check=`mysqlshow --user=root --password=$MYSQL_ROOT_PASSWORD | grep -ow "$MYSQL_DATABASE"`
 
if [ "$database_availability_check" == "$MYSQL_DATABASE" ]; then
exit 1
else
mysql -u root -p$MYSQL_ROOT_PASSWORD -e "grant all on *.* to 'root'@'%' identified by '$MYSQL_ROOT_PASSWORD';"
mysql -u root -p$MYSQL_ROOT_PASSWORD -e "create database $MYSQL_DATABASE;"
mysql -u root -p$MYSQL_ROOT_PASSWORD -e "grant all on $MYSQL_DATABASE.* to 'root'@'%' identified by '$MYSQL_ROOT_PASSWORD';"
supervisorctl stop database_creation && supervisorctl remove database_creation
echo "Database $MYSQL_DATABASE created"
fi

----------------------------------------------------------------------------------------------------------------------------------------------------------------------

cd magento/database_server

nano supervisord.conf 

#add this line super

[supervisord]
nodaemon=true


[program:mysql]
command=/bin/bash -c "touch /var/run/mysqld/mysqld.sock;touch /var/run/mysqld/mysqld.pid;chown -R mysql:mysql /var/lib/mysql;chown -R mysql:mysql /var/run/mysqld;chown -R mysql:mysql /var/log/>

[program:database_creation]
command=/bin/bash -c "chmod a+x /etc/mysql.sh; /etc/mysql.sh"

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#create magento2 dictory with in magento directory

magento$ mkdir magento2

magento$ cd magento2

magento2$ 

#install magento
 
sudo wget https://codeload.github.com/magento/magento2/zip/refs/tags/2.4.3
sudo apt install unzip
unzip 2.4.3
sudo mv magento2-2.4.3 /home/ubuntu/magento/magento2
sudo rm 2.4.3
sudo rm -rf magento2-2.4.3

--------------------------------------------------------------------------------------------------------------------------------------------------------------------

magento$

#Now run the command to build docker

magento$ docker-compose up -d

#Now go into the container of apache2 by using command

magento$ docker exec -it apache2 bash

#Run command composer and check the version of the composer if the composer is not installed the run the command for installing composer

/var/www/html/magento# composer

/var/www/html/magento# composer install

/var/www/html# mysql -u root -p -h 127.0.0.1

Enter password:password

mysql> show databases;

mysql> exit;

#Enable elasticsearch before installing the magento by command:

/etc/init.d/elasticsearch restart

apt install curl

curl -XGET 'http://localhost:9200'

/var/www/html/magento2# bin/magento setup:install --base-url=http://13.232.29.249/ --db-host=127.0.0.1 --db-name=magentodb --db-user=root --db-password=password --admin-firstname=admin --admin-lastname=admin --admin-email=admin@emai.com --admin-user=magentoadmin --admin-password=password@1 --language=en_US --currency=USD --timezone=America/Chicago --use-rewrites=1

/var/www/html/magento2# nano app/etc/env.php

#edit admin name

find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +

find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +

chown -R :www-data .

chmod -R 777 var/ setup/ vendor/ pub/ bin/

chmod u+x bin/magento

php bin/magento c:c

php bin/magento c:f

/etc/init.d/apache2 restart

nano /etc/apache2/sites-available/000-default.conf 

/etc/init.d/apache2 restart

#confiuration file

{
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/magento2/pub
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
}













