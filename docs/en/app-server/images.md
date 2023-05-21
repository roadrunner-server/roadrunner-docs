# App server — Docker Images

Following Docker images are available:

| Description                              | Links                                                                             | Status                                                                                                                                                                                                                   |
|------------------------------------------|-----------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Official RR image**                    | [Link](https://github.com/roadrunner-server/roadrunner/pkgs/container/roadrunner) | ![Latest Stable Version](https://img.shields.io/github/v/release/roadrunner-server/roadrunner.svg?maxAge=30) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) |
| **Third party image from `n1215`**       | [Link](https://github.com/n1215/roadrunner-docker-skeleton)                       | [![License](https://poser.pugx.org/n1215/roadrunner-docker-skeleton/license)](https://packagist.org/packages/n1215/roadrunner-docker-skeleton)                                                                           |
| **Third party image from `spacetab-io`** | [Link](https://github.com/spacetab-io/docker-roadrunner-php)                      | ![Latest Stable Version](https://img.shields.io/github/v/release/spacetab-io/docker-roadrunner-php) ![License](https://img.shields.io/github/license/spacetab-io/docker-roadrunner-php)                                  |


Here is an example of a `Dockerfile` that can be used to build a Docker image with RoadRunner for a PHP application:

```dockerfile
FROM php:8.2-cli-alpine3.17 as backend

RUN --mount=type=bind,from=mlocati/php-extension-installer:1.5,source=/usr/bin/install-php-extensions,target=/usr/local/bin/install-php-extensions \
     install-php-extensions opcache zip xsl dom exif intl pcntl bcmath sockets && \
     apk del --no-cache ${PHPIZE_DEPS} ${BUILD_DEPENDS}

WORKDIR /app

ENV COMPOSER_ALLOW_SUPERUSER=1
COPY --from=composer:2.3 /usr/bin/composer /usr/bin/composer

# Copy composer files from app directory to install dependencies
COPY ./app/composer.* .
RUN composer install --optimize-autoloader --no-dev

COPY --from=ghcr.io/roadrunner-server/roadrunner:2023.1.1 /usr/bin/rr /app

EXPOSE 8080/tcp

# Copy application files
COPY ./app .

# Run RoadRunner server
CMD ./rr serve -c .rr.yaml
```