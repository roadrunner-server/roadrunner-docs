# NGINX with RoadRunner

RoadRunner can operate alongside a web server, acting as a backend for processing PHP requests. To get started with using RoadRunner and NGINX within a Docker container, follow these steps:

1. **Create a `Dockerfile`**: Prepare a Dockerfile that includes RoadRunner. Be sure to install the necessary dependencies for each component:

```dockerfile
FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/roadrunner-server/roadrunner:latest as roadrunner
FROM --platform=${TARGETPLATFORM:-linux/amd64} php:8.1-alpine

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer
COPY --from=roadrunner /usr/bin/rr /usr/local/bin/rr
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

RUN mkdir /src
COPY worker.php /src
COPY .rr.yaml /src
COPY composer.json /src

WORKDIR /src

RUN apk update
RUN install-php-extensions sockets

RUN composer install

ENTRYPOINT ["rr"]
```

2. **Configure NGINX**: Create an nginx configuration file to configure NGINX as a reverse proxy for RoadRunner. Adjust the settings to match your application requirements:

```nginx configuration
server {
  listen 80;
  listen [::]:80;
  server_name RoadRunner;

  # http://roadrunner here is the DNS inside the docker
  location / {
    fastcgi_pass roadrunner:9000;
        # include the fastcgi_param setting
    include fastcgi_params; # <- much faster
#     proxy_pass http://roadrunner; # <- use this to use http proxy
    access_log off;
    error_log off;
#     proxy_set_header Host $host;
#     proxy_set_header X-Forwarded-For $remote_addr;
#     proxy_set_header X-Forwarded-Port $server_port;
#     proxy_set_header X-Forwarded-Host $host;
#     proxy_set_header X-Forwarded-Proto $scheme;
#     proxy_read_timeout 1200s;
  }

  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }
```

3. **Configure RoadRunner**: Create a `.rr.yaml` configuration file to specify how RoadRunner should interact with your PHP application:

```yaml
version: '3'

rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php worker.php"
  relay: pipes

http:
  address: 0.0.0.0:80
  pool:
    num_workers: 10
  fcgi:
    address: tcp://0.0.0.0:9000

logs:
  encoding: json
  level: error
  mode: production
```

4. **Create a PHP worker** to handle the HTTP requests:

```php
<?php
/**
 * @var Goridge\RelayInterface $relay
 */
use Spiral\Goridge;
use Spiral\RoadRunner;

ini_set('display_errors', 'stderr');
require __DIR__ . "/vendor/autoload.php";

$worker = RoadRunner\Worker::create();
$psr7 = new RoadRunner\Http\PSR7Worker(
    $worker,
    new \Nyholm\Psr7\Factory\Psr17Factory(),
    new \Nyholm\Psr7\Factory\Psr17Factory(),
    new \Nyholm\Psr7\Factory\Psr17Factory()
);

while ($req = $psr7->waitRequest()) {
    try {
        $resp = new \Nyholm\Psr7\Response();
        $resp->getBody()->write("Hello from the RoadRunner :)");

        $psr7->respond($resp);
    } catch (\Throwable $e) {
        $psr7->getWorker()->error((string)$e);
    }
}
```

And do not forget about the `composer.json`:

```json
{
  "minimum-stability": "dev",
  "prefer-stable": true,
  "require": {
    "spiral/roadrunner": "^2.0",
    "spiral/roadrunner-http": "^2.1",
    "spiral/roadrunner-worker": "^2.2",
    "spiral/goridge": "^3.2"
  }
}
```

5. **Create a docker-compose.yaml file**: To assemble and manage all components, create a `docker-compose.yaml` file that defines the RoadRunner and NGINX services, as well as their configurations:

```yaml
version: "3.8"

services:
  roadrunner:
    build:
      context: .
      dockerfile: Dockerfile
      # if needed to control the RR from the outside
    ports:
      - "127.0.0.1:6001:6001"
    command:
      - "serve"
      - "-c"
      - "/src/.rr.yaml"
    networks:
      nginx-docs:

  web:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
    volumes:
      - ./:/etc/nginx/conf.d
    environment:
      - NGINX_PORT=80
    networks:
      nginx-docs:

networks:
  nginx-docs:
    name: nginx-docs
```

# Tips and Best Practices

1. Consider using `fastcgi_pass` instead of `proxy_pass`: Using the `fastcgi_pass` directive might offer better performance in certain configurations.