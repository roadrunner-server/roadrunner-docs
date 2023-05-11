# RoadRunner — What is it?

RoadRunner is a high-performance PHP application server and process manager, designed with extensibility in mind through
its utilization of plugins. Developed in Go, RoadRunner operates by running your application in the form of workers,
where each worker represents an individual process, ensuring isolation and independence in their operation.

It is designed to be like a central processor for PHP applications, helping developers create faster, more
responsive and robust applications with ease.

![image](https://user-images.githubusercontent.com/773481/235296092-2f82643b-7822-4649-952a-0529efa3af88.png)

## Server

RoadRunner efficiently manages a collection of PHP processes, referred to as workers, and routes incoming requests from
various plugins to these workers. This communication is done through
the [goridge](https://github.com/roadrunner-server/goridge) protocol, enabling your PHP application to handle requests
and send responses back to clients.

![Base Diagram](https://user-images.githubusercontent.com/796136/65347341-79dd8600-dbe7-11e9-9621-1c5f2ef929e6.png)

The following plugins are designed to run workers and handle specific types of requests:

- [**HTTP**](../http/http.md) - Processes incoming HTTP requests from clients and forwards them to the PHP application.
- [**Jobs**](../queues/overview.md) - Handles queued tasks received from queue brokers and sends them to a consumer PHP
  application for processing.
- [**Centrifuge**](../plugins/centrifuge.md) - Manages events from Centrifugo WebSocket server clients and forwards them
  to the PHP application. It supports bidirectional communication, allowing for efficient and seamless interaction
  between the server and clients.
- [**gRPC**](../plugins/grpc.md) - Deals with gRPC requests from clients and passes them on to the PHP application.
- [**TCP**](../plugins/tcp.md) - Handles TCP requests from clients and routes them to the appropriate PHP application.
- [**Temporal**](../workflow/temporal.md) - Manages workflows and activities, allowing for the efficient handling of
  various tasks and processes.

By utilizing these plugins, RoadRunner ensures that your PHP application can effectively handle a wide range of requests
and communication protocols, delivering optimal performance and flexibility.

## RPC

RoadRunner also provides an RPC interface for communication between the application and the server, which plays a
significant role in enhancing the interaction between the two components. This interface is particularly useful when
working with the various plugins that support RPC communication, such as:

- [**KV**](../kv/overview.md) - A cache service that allows for efficient storage and retrieval of cached data.
- [**Locks**](../plugins/locks.md) - Offers a convenient means to manage distributed locks, ensuring resource access
  coordination across multiple processes or systems.
- [**Service**](../plugins/service.md) - A dynamic server processes supervisor that enables seamless management of
  server processes directly from the application.
- [**Jobs**](../queues/overview.md) - Provides the ability to dynamically manage queue pipelines from within the
  application, streamlining the execution of tasks and jobs.
- [**Logger**](../lab/logger.md) - Facilitates the forwarding of logs from the application to the RoadRunner logger,
  ensuring centralized and efficient log management.
- [**Metrics**](../lab/metrics.md) - Allows for the submission of application metrics to Prometheus, promoting
  comprehensive monitoring and analysis of application performance.

## PHP

RoadRunner keeps PHP workers alive between incoming requests. This means that you can completely eliminate bootload time
(such as framework initialization) and significantly speed up a heavy application.

![Base Diagram](https://user-images.githubusercontent.com/796136/65348057-00df2e00-dbe9-11e9-9173-f0bd4269c101.png)

Since a worker resides in memory, all open resources persist across requests, remaining accessible for subsequent
interactions. Utilizing the built-in [goridge](https://github.com/roadrunner-server/goridge), you can
offload complex computations to the application server. For instance, you can schedule a background PHP job or
even develop your own [custom RoadRunner plugin](../customization/plugin.md) to process complex tasks using Go, which
offers improved efficiency and performance. By leveraging the strengths of both PHP and Go, you can create a more robust
and high-performance solution for your applications.

It is worth mentioning the following properties of the PHP workers:

- **Isolation:** Each worker process operates independently, preventing interference with other worker processes and
  improving stability.
- **Scalability:** The shared-nothing architecture makes it easier to scale applications by simply adding more worker
  processes.
- **Fault Tolerance:** The failure of a single worker does not impact the functioning of other workers, ensuring
  uninterrupted service.
- **Simplified Development:** By maintaining the isolation of workers, the shared-nothing architecture reduces the
  complexity of managing shared resources and simplifies the development process.

## What's Next?

1. [PHP Workers](../php/worker.md) - Learn how to configure and run PHP workers.
2. [PHP Workers — RPC to RoadRunner](../php/rpc.md) - Learn how to use RPC to communicate with PHP applications.
3. [Writing a custom plugin](../customization/plugin.md) - Learn how to create a custom plugin.