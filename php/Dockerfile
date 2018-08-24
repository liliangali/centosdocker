FROM debian:jessie
LABEL maintainer "admin@centos.bz"
 
# 定义软件版本，编译工具，依赖等变量
ENV PHP_VERSION 5.6.30
ENV BUILD_TOOLS m4 \
    autoconf \
    autoconf2.13 \
    openssl \
    wget \
    gcc \
    make
 
ENV BUILD_DEPS libcurl4-gnutls-dev \
    libxml2-dev \
    zlib1g-dev \
    libpcre3-dev \
    libjpeg-dev \
    libpng12-dev \
    libfreetype6-dev \
    libmhash-dev \
    libmcrypt-dev \
    libssl-dev \
    libtool
 
ENV PHP_LOCATION /usr/local/php
ENV BUILD_ARG --prefix=${PHP_LOCATION} \
    --with-config-file-path=${PHP_LOCATION}/etc \
    --enable-fpm \
    --enable-bcmath \
    --with-pdo_sqlite \
    --with-gettext \
    --with-iconv \
    --enable-ftp \
    --with-sqlite3 \
    --enable-mbstring \
    --enable-sockets \
    --enable-zip \
    --enable-soap \
    --with-openssl \
    --with-zlib \
    --with-curl \
    --with-gd \
    --with-jpeg-dir \
    --with-png-dir \
    --with-freetype-dir \
    --with-mcrypt \
    --with-mhash \
    --with-mysql=mysqlnd \
    --with-mysqli=mysqlnd \
    --with-pdo-mysql=mysqlnd \
    --without-pear \
    --with-libdir=lib64 \
    --enable-opcache \
    --disable-cgi
 
ENV SRC_DIR /opt/php
 
WORKDIR ${SRC_DIR}
 
# 开始编译安装php
RUN apt-get update \
 && apt-get -y --no-install-recommends install ${BUILD_DEPS} ${BUILD_TOOLS} \
 && wget http://php.net/distributions/php-${PHP_VERSION}.tar.gz \
 && tar xf php-${PHP_VERSION}.tar.gz \
 && cd php-${PHP_VERSION} \
 && ln -s /usr/lib/x86_64-linux-gnu/libssl.so /usr/lib/libssl.so \
 && ln -s /usr/lib /usr/lib64 \
 && ./configure ${BUILD_ARG} \
 && make -j$(nproc) \
 && make install \
 && cp php.ini-production ${PHP_LOCATION}/etc/php.ini \
 && cp ${PHP_LOCATION}/etc/php-fpm.conf.default ${PHP_LOCATION}/etc/php-fpm.conf \
 && rm -rf ${SRC_DIR} \
 && apt-get purge -y --auto-remove ${BUILD_TOOLS} \
 && rm -rf /var/lib/apt/lists/*
 
 
WORKDIR ${PHP_LOCATION}/etc/
 
# 配置php-fpm，即使用sed工具编辑php-fpm.conf和php.ini文件，这里的php-fpm相关配置命令不与上面的编译命令合在一起来减小层数是因为
# 配置文件可能会改动比较多，这样分开当配置文件更改时可以直接使用缓存跳过编译步骤，加快构建速度。
RUN set_php_variable(){ \
  local key=$1; \
  local value=$2; \
  if grep -q -E "^$key\s*=" php.ini;then \
   sed -i -r "s#^$key\s*=.*#$key=$value#" php.ini; \
  else \
   sed -i -r "s#;\s*$key\s*=.*#$key=$value#" php.ini; \
  fi; \
  if ! grep -q -E "^$key\s*=" php.ini;then \
   echo "$key=$value" >> php.ini; \
  fi; \
 } \
 && BASE_DIR=/home/docker/php \
 && set_php_variable disable_functions "dl,eval,assert,exec,popen,system,passthru,shell_exec,escapeshellarg,escapeshellcmd,proc_close,proc_open" \
 && set_php_variable expose_php Off \
 && set_php_variable error_log ${BASE_DIR}/php_errors.log \
 && set_php_variable request_order "CGP" \
 && set_php_variable cgi.fix_pathinfo 0 \
 && set_php_variable short_open_tag on \
 && set_php_variable date.timezone Asia/Chongqing \
 && sed -i 's/^user =.*/user = www-data/' php-fpm.conf \
 && sed -i 's/^group =.*/group = www-data/' php-fpm.conf \
 && sed -i "s#;slowlog = log/\$pool.log.slow#slowlog = ${BASE_DIR}/\$pool.log.slow#" php-fpm.conf \
 && sed -i 's/;request_slowlog_timeout = 0/request_slowlog_timeout = 5/' php-fpm.conf \
 && sed -i 's/^pm.max_children.*/pm.max_children =20/' php-fpm.conf \
 && sed -i 's/^pm.start_servers.*/pm.start_servers =5/' php-fpm.conf \
 && sed -i 's/^pm.min_spare_servers.*/pm.min_spare_servers =3/' php-fpm.conf \
 && sed -i 's/^pm.max_spare_servers.*/pm.max_spare_servers =8/' php-fpm.conf \
 && sed -i '/\[global\]/a\daemonize =no' php-fpm.conf \
 && sed -i 's/^listen.*/listen =0.0.0.0:9000/' php-fpm.conf \
 && echo "[opcache]\n \
   zend_extension=opcache.so\n \
   opcache.memory_consumption=128\n \
   opcache.interned_strings_buffer=8\n \
   opcache.max_accelerated_files=4000\n \
   opcache.revalidate_freq=60\n \
   opcache.fast_shutdown=1 \n" >> php.ini
 
ENTRYPOINT ["/usr/local/php/sbin/php-fpm"]
 
EXPOSE 9000
