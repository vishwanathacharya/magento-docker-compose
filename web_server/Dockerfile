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
