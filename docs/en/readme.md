# Table of Contents

RoadRunner is an open-source (MIT licensed), high-performance PHP application server, load balancer, and process
manager.

![CI](https://github.com/spiral/roadrunner-docs/workflows/CI/badge.svg)

> Documentation for v1.0 of RoadRunner is
> available [here](https://github.com/roadrunner-server/roadrunner-docs/tree/1.0).

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
    * [Restarting](php/restarting.md)
    * [Process Supervisor](php/supervisor.md)
    * [RPC to App Server](php/rpc.md)
    * [Caveats](php/limitations.md)
    * [Debugging](php/debugging.md)
* App Server
    * [CLI Commands](app-server/cli.md)
    * [Production Usage Tips](app-server/production.md)
    * [Writing a RR systemd unit file](app-server/systemd.md)
    * [Healthcheck](app-server/health.md)
    * [Jobs (queues)](plugins/jobs.md) 
    * [RR with custom plugins](app-server/build.md)
    * [AWS Lambda](app-server/aws-lambda.md)
    * [RR Events Bus](app-server/events-bus.md)
    * [Overriding HTTP error codes](app-server/http-error-codes.md)
    * [gRPC](app-server/grpc.md)
    * [HTTPS](app-server/https.md)
    * [Access-Logs](app-server/access-logs.md)
    * [PubSub](app-server/pubsub.md)
* Plugins
    * [What is it?](plugins/intro.md)
    * [Write a plugin](plugins/plugin.md)
    * [HTTP (transport)](plugins/http.md)
    * [JOBS (queues)](plugins/jobs.md)
    * [Key-Value](plugins/kv.md)
    * [Configuration](plugins/config.md)
    * [gRPC](plugins/grpc.md)
    * [Informer](plugins/informer.md)
    * [Broadcast](plugins/broadcast.md)
    * [Logger](plugins/logger.md)
    * [Metrics (prometheus)](plugins/metrics.md)
    * [Reload](plugins/reload.md)
    * [Resetter](plugins/resetter.md)
    * [RPC](plugins/rpc.md)
    * [Server](plugins/server.md)
    * [Service (like systemd)](plugins/service.md)
    * [Status](plugins/status.md)
    * [Websockets](plugins/websocket.md)
    * [TCP](plugins/tcp.md)
    * [Fileserver](plugins/fileserver.md)
* HTTP Middleware
    * [Writing a middleware](middleware/writing-a-middleware.md)
    * [Headers](middleware/headers.md)
    * [Gzip](middleware/gzip.md)
    * [Static](middleware/static.md)
    * [Sendfile](middleware/sendfile.md)
    * [Newrelic](middleware/newrelic.md)
    * [Cache](middleware/cache.md)
    * [OTEL](middleware/otel.md)
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
    * [CodeIgniter](integration/codeigniter.md)
    * [**All Composer Libraries**](https://packagist.org/packages/spiral/roadrunner/dependents)
* Docker
    * [Ports and Containers](docker/ports.md)
    * [Available Images](docker/images.md)
* Errors
    * [CRC validation](known-issues/stdout-crc.md)
    * [Allocate timeout](known-issues/allocate-timeout.md)

