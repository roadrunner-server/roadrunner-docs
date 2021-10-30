# Table of Contents

RoadRunner is an open-source (MIT licensed), high-performance PHP application server, load balancer, and process
manager.

![CI](https://github.com/spiral/roadrunner-docs/workflows/CI/badge.svg)

> Documentation for v1.0 of RoadRunner is available [here](https://github.com/spiral/roadrunner-docs/tree/1.0).

* Introduction
    * [What is it?](intro/about.md)
    * [Features](intro/features.md)
    * [Installation](intro/install.md)
    * [Configuration Reference](intro/config.md)
    * [LICENSE](license.md)
* PHP Workers
    * [Worker](php/worker.md)
    * [Environment](php/environment.md)
    * [Developer Mode](php/developer.md)
    * [Error Handling](php/error-handling.md)
    * [Restarting](php/restarting.md)
    * [Process Supervisor](php/limit.md)
    * [RPC to App Server](php/rpc.md)
    * [Caveats](php/caveats.md)
    * [Debugging](php/debugging.md)
* HTTPs and HTTP/2
    * [HTTPs and HTTP/2](http/https.md)
    * [Static Content](http/static.md)
    * [HTTP error codes](beep-beep/http-error-codes.md)
    * [Headers](http/headers.md)
    * [Access logs](http/access-logs.md)
    * [Available middleware](http/available-middleware.md)
    * [Golang Middleware](http/writing-a-middleware.md)
* App Server
    * [CLI Commands](beep-beep/cli.md)
    * [Logging](beep-beep/logging.md)
    * [Auto-Reloading](beep-beep/reload.md)
    * [Production Usage](beep-beep/production.md)
    * [Writing a RR systemd unit file](beep-beep/systemd.md)
    * [Prometheus Metrics](beep-beep/metrics.md)
    * [Healthcheck](beep-beep/health.md)
    * [Building a Server](beep-beep/build.md)
    * [KeyValue Storage](beep-beep/kv.md)
    * [RPC](beep-beep/rpc.md)
    * [Jobs](beep-beep/jobs.md)
    * [GRPC](beep-beep/grpc.md)
    * [Write a Plugin](beep-beep/plugin.md)
    * [Service plugin](beep-beep/service.md)
    * [AWS Lambda](library/aws-lambda.md)
    * [RR Events Bus](library/events-bus.md)
* Workflow Engine
    * [About Temporal.IO](workflow/temporal.md)
    * [Worker](workflow/worker.md)
* Integrations V1
    * [Migration from V1 to V2](integration/migration.md)
    * [CakePHP](integration/cake.md)
    * [Laravel](integration/laravel.md)
    * [Slim](integration/slim.md)
    * [Spiral Framework](integration/spiral.md)
    * [Symfony Framework](integration/symfony.md)
    * [Symlex Framework](integration/symlex.md)
    * [Ubiquity Framework](integration/ubiquity.md)
    * [Zend Expressive](https://github.com/sergey-telpuk/roadrunner-zend-expressive-integration)
    * [Yii2 and Yii3](integration/yii.md)
    * [Phalcon3 and Phalcon4](integration/phalcon.md)
    * [Mezzio](integration/mezzio.md)
    * [Chubbyphp Framework](integration/chubbyphp.md)
    * [**All Composer Libraries**](https://packagist.org/packages/spiral/roadrunner/dependents)
* Docker
    * [Ports and Containers](docker/ports.md)
    * [Available Images](docker/images.md)
