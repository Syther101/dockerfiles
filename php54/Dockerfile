FROM datadoglt/base:latest
MAINTAINER "DataDog" <info@datadog.lt>

ENV GITHUB_TOKEN 89fdb82a7b2c50bf20c39bfb87614acd68901e80
ENV WWW_DIR /var/www

COPY entrypoint.sh /entrypoint.sh
COPY init.sh /init.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["fpm"]
WORKDIR $WWW_DIR

EXPOSE 9000

# install php with packages
# if soap extension is missing it fails to launch fpm
# related to soap wsdl cache dir in www.conf
RUN yum install -y \
        git \
        php-bcmath \
        php-cli \
        php-mcrypt \
        php-curl \
        php-fpm \
        php-gd \
        php-iconv \
        php-intl \
        php-mysql \
        php-mbstring \
        php-pdo \
        php-process \
        php-soap \
        php-devel \
        php-pear \
        make \
        gcc \
        gcc-c++ \
        autoconf \
        automake \
        php-xml && \
    yum clean all

RUN pecl install Xdebug

# change permissions for default php session and wsdl cache directories
RUN mkdir -p /var/lib/php/session && \
    mkdir -p /var/lib/php/wsdlcache && \
    chown www:www -R /var/lib/php/session && \
    chown www:www -R /var/lib/php/wsdlcache

# set custom configuration
RUN echo "date.timezone=UTC" > /etc/php.d/40-timezone.ini && \
    echo "session.save_path=/var/lib/php/session" > /etc/php.d/40-sessions.ini && \
    echo "soap.wsdl_cache_dir=/var/lib/php/wsdlcache" > /etc/php.d/40-soap.ini && \
    echo "cgi.fix_pathinfo=0" > /etc/php.d/40-cgi.ini && \
    echo "zend_extension=/usr/lib64/php/modules/xdebug.so;" > /etc/php.d/40-xdebug.ini && \
    echo "display_errors=stderr" > /etc/php.d/30-errors.ini

# copy default configuration of fpm
COPY www.conf /etc/php-fpm.d/www.conf

# install composer
RUN curl -sS https://getcomposer.org/installer | php -- \
        --install-dir=/usr/bin \
        --filename=composer && \
    composer config -g github-oauth.github.com $GITHUB_TOKEN
