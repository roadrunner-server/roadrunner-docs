# Table of Contents

RoadRunner is an open source (MIT licensed), high-performance PHP application server, load balancer and process manager. 
It supports running as a service with the ability to extend its functionality on a per-project basis. RoadRunner includes PSR-7 compatible HTTP server.

* Introduction
   * [About RoadRunner](intro/about.md)
   * [Installation](intro/install.md)
   * [Configuration](intro/config.md)
   * [LICENSE](license.md)
* Using RoadRunner
   * [Environment Configuration](usage/environment.md)
   * [HTTPS and HTTP/2](usage/https.md)
   * [Headers](usage/headers.md)
   * [PHP Workers](usage/php-workers.md)
   * [Caveats](usage/caveats.md)
   * [Debugging](usage/debugging.md)
   * [Server Commands](usage/server-commands.md)
   * [RPC Integration](usage/rpc.md)
   * [Restarting Workers](usage/restarting-workers.md)
   * [IDE integration](usage/ide-integration.md)
   * [Error Handling](usage/error-handling.md)
   * [Application Metrics](usage/metrics.md)
   * [Production Usage](usage/production.md)
* Integrations
   * [Symfony Framework](integrations/symfony.md)
   * [Laravel Framework](https://github.com/spiral/roadrunner/wiki/Laravel-Framework)
   * [Slim Framework](https://github.com/spiral/roadrunner/issues/62)
   * [Yii2/3 Framework](https://github.com/spiral/roadrunner/issues/78) (in progress)
   * [Zend Expressive](https://github.com/sergey-telpuk/roadrunner-zend-expressive-integration)
   * [CakePHP](https://github.com/CakeDC/cakephp-roadrunner)
   * [**All Integrations**](https://packagist.org/packages/spiral/roadrunner/dependents) 
* Server Customization
   * [Building Server](server/building-server.md)
   * [Writing Services](server/writing-services.md)
   * [HTTP Middleware](server/middleware.md)
* As Library
   * [Event Listeners](library/event-listeners.md)
   * [Standalone Usage](library/standalone-usage.md)
   * [AWS Lambda](library/aws-lambda.md)
