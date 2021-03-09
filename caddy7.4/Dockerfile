FROM caddy/caddy:2.3.0-builder-alpine AS builder
LABEL maintainer "Daniel Steiner <kontakt@danielsteiner.net>"
LABEL org.opencontainers.image.version=v2.1.1-php7.4
LABEL org.opencontainers.image.title=Caddy
LABEL org.opencontainers.image.description="a powerful, enterprise-ready, open source web server with automatic HTTPS written in Go - now with PHP 7.4"
LABEL org.opencontainers.image.url=https://danielsteiner.net/docker/images/caddy
LABEL org.opencontainers.image.documentation=https://danielsteiner.net/docker/docs/caddy
LABEL org.opencontainers.image.vendor="Daniel Steiner Web- & Appentwicklung"
LABEL org.opencontainers.image.licenses=Apache-2.0
LABEL org.opencontainers.image.source="https://github.com/danielsteiner/caddy-docker-php"

ARG PUID="1000"
ARG PGID="1000"

RUN xcaddy build \
  --with github.com/abiosoft/caddy-exec \
  --with github.com/abiosoft/caddy-hmac \
  --with github.com/aksdb/caddy-cgi/v2 \
  --with github.com/caddy-dns/cloudflare \
  --with github.com/caddyserver/ntlm-transport \
  --with github.com/chukmunnlee/caddy-openapi \
  --with github.com/greenpau/caddy-auth-jwt \
  --with github.com/greenpau/caddy-auth-portal \
  --with github.com/greenpau/caddy-trace \
  --with github.com/hairyhenderson/caddy-teapot-module \
  --with github.com/lolPants/caddy-requestid \
  --with github.com/mholt/caddy-webdav \
  --with github.com/porech/caddy-maxmind-geolocation

FROM php:7.4-fpm-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy

RUN apk --no-cache update \
  && apk --no-cache upgrade \
  && apk add --no-cache --virtual \
  .build-deps \
  $PHPIZE_DEPS \
  gcc \
  g++ \
  autoconf \
  tar \
  nodejs \ 
  npm \
  nodejs-npm \
  yarn \
  && apk add --no-cache \
  libxslt-dev \
  tidyhtml-dev \
  net-snmp-dev \
  aspell-dev \
  freetds-dev \
  openldap-dev \
  gettext-dev \
  imap-dev \
  openssh \
  sudo \
  make \
  shadow \
  libmcrypt-dev \
  gmp-dev \
  openssl \
  mariadb-client \
  curl \
  freetype \
  libpng \
  libjpeg-turbo \
  freetype-dev \
  libpng-dev \
  libjpeg-turbo-dev \
  libwebp-dev \
  libzip-dev \
  bzip2-dev \
  postgresql-dev \
  supervisor \
  bash \
  nginx \
  pngquant \
  jpegoptim \
  zip \
  icu-dev \
  libxml2-dev \
  dcron \
  wget \
  rsync \
  ca-certificates \
  oniguruma-dev \
  && phpModules=" \
  bcmath \
  bz2 \
  calendar \
  exif \
  gd \
  gettext \
  gmp \
  imap \
  intl \
  ldap \
  mysqli \
  pcntl \
  pdo_dblib \
  pdo_mysql \
  pdo_pgsql \
  pgsql \
  pspell \
  shmop \
  snmp \
  soap \
  sockets \
  sysvmsg \
  sysvsem \
  sysvshm \
  tidy \
  xsl \
  " \
  && NPROC=$(getconf _NPROCESSORS_ONLN) \
  && docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg --with-webp \
  #    Install APCU
  #    @see https://github.com/docker-library/php/issues/1029
  && mkdir -p /usr/src/php/ext/apcu  \
  && mkdir -p /usr/src/php/ext/amqp  \
  && mkdir -p /usr/src/php/ext/igbinary  \
  && mkdir -p /usr/src/php/ext/mongodb  \
  && mkdir -p /usr/src/php/ext/redis  \
  && curl -fsSL https://pecl.php.net/get/apcu | tar xvz -C "/usr/src/php/ext/apcu" --strip 1 \
  && curl -fsSL https://pecl.php.net/get/amqp | tar xvz -C "/usr/src/php/ext/amqp" --strip 1 \
  && curl -fsSL https://pecl.php.net/get/igbinary | tar xvz -C "/usr/src/php/ext/igbinary" --strip 1 \
  && curl -fsSL https://pecl.php.net/get/mongodb | tar xvz -C "/usr/src/php/ext/mongodb" --strip 1 \
  && curl -fsSL https://pecl.php.net/get/redis | tar xvz -C "/usr/src/php/ext/redis" --strip 1 \
  && docker-php-ext-install -j${NPROC} $phpModules \
  && apk del --no-cache gcc g++ freetype-dev libpng-dev libjpeg-turbo-dev .build-deps

RUN addgroup -g 1000 www-user && \
  adduser -D -H -u 1000 -G www-user www-user && \
  id -u www-user && \
  id -G www-user && \ 
  sed -i "s|^user = .*|user = www-user|g" /usr/local/etc/php-fpm.d/www.conf && \
  sed -i "s|^group = .*|group = www-user|g" /usr/local/etc/php-fpm.d/www.conf

# Composer
RUN curl --silent --show-error --fail --location \
  --header "Accept: application/tar+gzip, application/x-gzip, application/octet-stream" \
  "https://getcomposer.org/installer" \
  | php -- --install-dir=/usr/bin --filename=composer

RUN caddy version && \
  caddy list-modules  && \
  echo "clear_env = no" >> /usr/local/etc/php-fpm.conf && \
  php -v  && \
  php-fpm -D

EXPOSE 80 443 2019 2020
WORKDIR /etc/caddy/

COPY ./supervisord.conf /etc/supervisord.conf
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]