# App server — Nginx with RoadRunner

RoadRunner seamlessly integrates with various web servers like Nginx, providing a powerful backend solution for
processing PHP requests.

## Nginx configuration

### FastCGI

RoadRunner can be configured to listen for FastCGI requests on a specific port. (Disabled by default.)

```yaml .rr.yaml
version: "3"

http:
  fcgi:
    address: tcp://0.0.0.0:9000
```

The FastCGI method allows Nginx to communicate directly with the RoadRunner server using the FastCGI protocol. This
method is suitable when both Nginx and RoadRunner are running on the same machine.

> **Warning**
> Remember to adjust the configuration examples according to your specific environment and requirements. If RoadRunner
> and Nginx are running in separate Docker containers, utilize the container DNS names (e.g., `roadrunner:9000`) instead
> of IP addresses in the Nginx configuration.

```nginx docker/nginx/rr.conf
server {
   listen 80;
   listen [::]:80;
   server_name _;
   
   location / {
      fastcgi_pass 127.0.0.1:9000;
      include fastcgi_params;

      access_log off;
      error_log off;
   }
}
```

> **Note**
> Consider using `fastcgi_pass` instead of `proxy_pass`: Using the `fastcgi_pass` directive might offer better
> performance in certain configurations.

### Proxy

RoadRunner can be configured to listen for HTTP requests on a specific port.

```yaml .rr.yaml
http:
  address: 0.0.0.0:8080
```

> **Note**
> Read more about configuring HTTP server in the [HTTP Plugin](../http/http.md) section.

The Proxy method involves configuring Nginx to act as a reverse proxy for RoadRunner. Nginx receives client requests and
forwards them to RoadRunner for processing. This method is useful when both are running on separate machines or when
additional load balancing or caching features are required.

> **Warning**
> Remember to adjust the configuration examples according to your specific environment and requirements. If RoadRunner
> and Nginx are running in separate Docker containers, utilize the container DNS names (e.g., `roadrunner:8080`) instead
> of IP addresses in the Nginx configuration.

```nginx docker/nginx/rr.conf
server {
   listen 80;
   listen [::]:80;
   server_name _;
   
   location / {
      proxy_pass http://127.0.0.1:8080;
      proxy_set_header Host $host;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header X-Forwarded-Port $server_port;
      proxy_set_header X-Forwarded-Host $host;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_read_timeout 1200s;
}
```

### WebSocket proxy

To enable WebSocket connections using Nginx proxy, you need to configure the proxy accordingly.

This can be done by including the following configuration in the Nginx configuration file:

```nginx docker/nginx/rr.conf
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
   listen 80;
   listen [::]:80;
    server_name _;

    location /connection/websocket {
        proxy_pass http://127.0.0.1:8000/connection/websocket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
    }
    
    location / {
        proxy_pass http://127.0.0.1:9000;
        # ...
    }
}
```

> **Warning**
> `http://127.0.0.1:8000` is the default address for the Centrifugo WebSocket server and `/connection/websocket` is the
> default path for Bidirectional WebSocket connections. 

The location `/connection` block defines the path where WebSocket connections will be handled.

## Docker

In this example, we will demonstrate how to use RoadRunner with Nginx in a Docker environment.

### Dockerfile

```docker docker/app/Dockerfile
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

### RoadRunner configuration

Create a `.rr.yaml` configuration file to specify how RoadRunner should interact with your PHP application

```yaml .rr.yaml
version: '3'

rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php worker.php"
  relay: pipes

http:
  address: 0.0.0.0:8080
  fcgi:
    address: tcp://0.0.0.0:9000
  pool:
    num_workers: 10

logs:
  encoding: json
  level: error
  mode: production
```

### PHP Worker

Create a PHP worker to handle the HTTP requests.

**Here is a simple example:**

```php worker.php
<?php
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

And do not forget about the `composer.json` file:

```json composer.json
{
  "minimum-stability": "dev",
  "prefer-stable": true,
  "require": {
    "spiral/roadrunner-http": "^3.0",
    "spiral/goridge": "^4.0"
  }
}
```

> **Note**
> Read more about the RoadRunner PHP Worker in the [PHP Workers](../php/worker.md) section.

### Docker Compose

To assemble and manage all components, create a `docker-compose.yaml` file that defines the RoadRunner and Nginx
services, as well as their configurations

```yaml docker-compose.yaml
version: "3.8"

services:
  roadrunner:
    build:
      context: .
      dockerfile: docker/app/Dockerfile
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
      - ./docker/nginx:/etc/nginx/conf.d
    environment:
      - NGINX_PORT=80
    networks:
      nginx-docs:

networks:
  nginx-docs:
    name: nginx-docs
```

> **Note**
> Store one of the configuration files provided in the [Nginx configuration](#nginx-configuration) section in
> the `docker/nginx` directory.