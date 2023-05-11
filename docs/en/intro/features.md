# RoadRunner — Features

RoadRunner is a highly performant and flexible HTTP/HTTP2 server that is fully compatible
with [PSR-7](https://www.php-fig.org/psr/psr-7/)/[PSR-17](https://www.php-fig.org/psr/psr-17/) standards.

It is designed to replace traditional Nginx+FPM setups with enhanced performance and versatility. With its wide
range of features, RoadRunner is a production-ready solution that is PCI DSS compliant, ensuring the security and safety
of your web applications.

**Here is a list of the features that make RoadRunner a great choice for your web application server:**

- Production-ready
- PCI DSS compliant (HTTP plugin)
- PSR-7 HTTP server (file uploads, error handling, static files, hot reload, middleware, event listeners)
- HTTPS and HTTP/2 support (including HTTP/2 Push, H2C)
- A fully customizable http(s)/2 server
- FastCGI support (HTTP plugin)
- Flexible environment configuration
- No external PHP dependencies (64bit version required)
- Integrated metrics (Prometheus)
- [Workflow engine](https://github.com/temporalio/sdk-php) by [Temporal.io](https://temporal.io)
- Websockets support by [Centrifugo](https://centrifugal.dev/) websocket server
- OpenTelemetry support
- Works over TCP, UNIX sockets and process pipes
- Automatic worker replacement, graceful and safe PHP process destruction
- Worker create/allocate/destroy timeouts
- Max requests per worker limitation
- Worker lifecycle management (controller)
    - `max_memory` (graceful stop)
    - `ttl` (graceful stop)
    - `idle_ttl` (graceful stop)
    - `exec_tll` (brute, max_execution_time)
- Protocol, worker and job level error management (including PHP errors)
- Development Mode
- Application server for [Spiral](https://github.com/spiral/framework)
- Integration with popular PHP frameworks such as
    - [Symfony](https://github.com/php-runtime/roadrunner-symfony-nyholm),
    - [Laravel](https://github.com/laravel/octane),
    - Slim,
    - CakePHP
- Compatible with both Windows and WSL2, with support for Unix sockets (`AF_UNIX`) on Windows 10/11.

The list of features mentioned above is just the tip of the iceberg. RoadRunner is actively developed by maintainers
and contributors, which means new features are constantly being added.

If you have a feature request in mind, you can check
out [Github issues](https://github.com/roadrunner-server/roadrunner/issues) page. Here you'll find a list of open
feature requests. The RoadRunner community is active and responsive, so feel free to join the discussion on
our [Discord channel](https://discord.gg/spiralphp) or [contribute](./contributing.md) to the project.

<a href="https://discord.gg/spiralphp"><img src="https://img.shields.io/badge/discord-chat-magenta.svg"></a>

With the support of the community, RoadRunner will continue to grow and evolve to meet the needs of modern web
development.