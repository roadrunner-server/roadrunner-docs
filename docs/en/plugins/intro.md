# Plugins — What is it?

RoadRunner boasts a powerful plugin system that not only comes with an extensive suite of pre-built plugins, but also
allows developers to create their own custom plugins and share them with the community. This versatility makes
RoadRunner an adaptable and flexible solution for a wide range of applications and use cases.

## Available plugins

RoadRunner offers numerous plugins out of the box, designed to cater to common requirements and streamline the
development process. Some of the most notable plugins include:

- [**Centrifuge**](./centrifuge.md): Real-time websocket messaging and broadcasting.
- [**HTTP**](../http/http.md): Efficient and scalable HTTP server implementation.
- [**Logger**](../lab/logger.md): Robust logging capabilities for various output formats and destinations.
- [**App-Logger**](../lab/applogger.md): Send application logs to the RoadRunner logger from PHP application.
- [**gRPC**](./grpc.md): Efficient and scalable gRPC server implementation.
- [**Temporal**](../workflow/temporal.md): Workflow and task orchestration with distributed computing capabilities.
- [**Server**](./server.md): Core server functionality and lifecycle management.
- [**Service**](./service.md): Start and monitor services like a supervisor.
- [**Locks**](./locks.md): Distributed locking mechanisms for concurrency control.
- [**TCP**](./tcp.md): High-performance TCP server for custom networking solutions.
- [**Metrics**](../lab/metrics.md): Application-level metrics and monitoring.
- [**KV**](../kv/overview.md): Key-value store interface for storage and retrieval of data.
- [**Jobs**](../queues/overview.md): Background job processing and management.
- [**HealthChecks**](../lab/health.md): Health monitoring and reporting for system components.
- [**OpenTelemetry (otel)**](../lab/otel.md): Distributed tracing and observability with OpenTelemetry integration.

## Custom Plugins

In addition, RoadRunner encourages developers to create their own custom plugins, tailored to meet specific requirements
or extend the core functionality. By creating plugins in Go, developers can tap into the performance, concurrency, and
scalability benefits that Go offers, while still using PHP for the core application logic.

To get started with custom plugin development, refer to [Writing Plugins](../customization/plugin.md)
and [HTTP Middleware](../customization/middleware.md) documentation. It provides a comprehensive guide on how to
create, test, and integrate your own plugins into RoadRunner.

## Community Sharing

RoadRunner fosters a collaborative ecosystem by encouraging developers to share their custom plugins with the community.
By sharing your work, you contribute to a growing library of plugins, which in turn helps other developers find
solutions to their challenges and fosters innovation.

If you've developed a custom plugin that you believe would benefit others, consider submitting it to the RoadRunner
Plugin Repository, where it can be discovered and used by the broader community.
