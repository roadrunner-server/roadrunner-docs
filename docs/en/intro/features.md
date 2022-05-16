<p align="center">
 <img src="https://user-images.githubusercontent.com/796136/50286124-6f7f3780-046f-11e9-9f45-e8fedd4f786d.png" height="75px" alt="RoadRunner">
</p>
<p align="center">
 <a href="https://packagist.org/packages/spiral/roadrunner"><img src="https://poser.pugx.org/spiral/roadrunner/version"></a>
	<a href="https://pkg.go.dev/github.com/roadrunner-server/roadrunner/v2?tab=doc"><img src="https://godoc.org/github.com/roadrunner-server/roadrunner/v2?status.svg"></a>
    <a href="https://codecov.io/gh/roadrunner-server/roadrunner/"><img src="https://codecov.io/gh/roadrunner-server/roadrunner/branch/master/graph/badge.svg"></a>
	<a href="https://github.com/roadrunner-server/roadrunner/actions"><img src="https://github.com/roadrunner-server/roadrunner/workflows/rr_cli_tests/badge.svg" alt=""></a>
    <a href="https://codecov.io/gh/roadrunner-server/rr-e2e-tests/"><img src="https://codecov.io/gh/roadrunner-server/rr-e2e-tests/branch/master/graph/badge.svg"></a>
    <a href="https://github.com/roadrunner-server/rr-e2e-tests/actions"><img src="https://github.com/roadrunner-server/rr-e2e-tests/workflows/linux_stable/badge.svg" alt=""></a>
	<a href="https://github.com/roadrunner-server/rr-e2e-tests/actions"><img src="https://github.com/roadrunner-server/rr-e2e-tests/workflows/Linters/badge.svg" alt=""></a>
	<a href="https://goreportcard.com/report/github.com/roadrunner-server/roadrunner"><img src="https://goreportcard.com/badge/github.com/roadrunner-server/roadrunner"></a>
	<a href="https://lgtm.com/projects/g/roadrunner-server/roadrunner/alerts/"><img alt="Total alerts" src="https://img.shields.io/lgtm/alerts/g/roadrunner-server/roadrunner.svg?logo=lgtm&logoWidth=18"/></a>
	<a href="https://discord.gg/TFeEmCs"><img src="https://img.shields.io/badge/discord-chat-magenta.svg"></a>
	<a href="https://packagist.org/packages/spiral/roadrunner"><img src="https://img.shields.io/packagist/dd/spiral/roadrunner?style=flat-square"></a>
    <img alt="All releases" src="https://img.shields.io/github/downloads/roadrunner-server/roadrunner/total">
</p>

RoadRunner is an open-source (MIT licensed) high-performance PHP application server, load balancer, and process manager.
It supports running as a service with the ability to extend its functionality on a per-project basis.

RoadRunner includes PSR-7/PSR-17 compatible HTTP and HTTP/2 server and can be used to replace classic Nginx+FPM setup
with much greater performance and flexibility.

## Join our discord server: [Link](https://discord.gg/TFeEmCs)

<p align="center">
	<a href="https://roadrunner.dev/"><b>Official Website</b></a> |
	<a href="https://roadrunner.dev/docs"><b>Documentation</b></a> |
    <a href="https://github.com/orgs/spiral/projects/2"><b>Release schedule</b></a>
</p>

Features:
--------
- Production-ready
- PCI DSS compliant
- PSR-7 HTTP server (file uploads, error handling, static files, hot reload, middlewares, event listeners)
- HTTPS and HTTP/2 support (including HTTP/2 Push, H2C)
- A Fully customizable server, FastCGI support
- Flexible environment configuration
- No external PHP dependencies (64bit version required), drop-in (based on [Goridge](https://github.com/roadrunner-server/goridge))
- Load balancer, process manager and task pipeline
- Integrated metrics (Prometheus)
- [Workflow engine](https://github.com/temporalio/sdk-php) by [Temporal.io](https://temporal.io)
- Works over TCP, UNIX sockets and standard pipes
- Automatic worker replacement and safe PHP process destruction
- Worker create/allocate/destroy timeouts
- Max jobs per worker
- Worker lifecycle management (controller)
  - maxMemory (graceful stop)
  - TTL (graceful stop)
  - idleTTL (graceful stop)
  - execTTL (brute, max_execution_time)
- Payload context and body
- Protocol, worker and job level error management (including PHP errors)
- Development Mode
- Integrations with Symfony, [Laravel](https://github.com/spiral/roadrunner-laravel), Slim, CakePHP, Zend Expressive
- Application server for [Spiral](https://github.com/spiral/framework)
- Included in Laravel Octane
- Automatic reloading on file changes
- Works on Windows (Unix sockets (AF_UNIX) supported on Windows 10)

License:
--------
The MIT License (MIT). Please see [`LICENSE`](./LICENSE) for more information. Maintained
by [Spiral Scout](https://spiralscout.com).

## Contributors

Thanks to all the people who already contributed!

<a href="https://github.com/roadrunner-server/roadrunner/graphs/contributors">
  <img src="https://contributors-img.web.app/image?repo=roadrunner-server/roadrunner" />
</a>