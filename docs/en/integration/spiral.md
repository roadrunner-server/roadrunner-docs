# Integration — Spiral Framework

[Spiral Framework](https://spiral.dev) is a robust and powerful PHP framework developed by the R&D team
at [Spiral Scout](https://spiralscout.com/). It is designed to facilitate the development and maintenance of medium to
large-sized enterprise applications. 

Spiral prioritizes developer experience and offers an intuitive and user-friendly environment, akin to popular 
frameworks like Laravel and Symfony. **One of the core strengths of Spiral is its efficient memory management and 
prevention of memory leaks through advanced techniques.**

RoadRunner is seamlessly integrated with Spiral to enhance the overall performance and scalability of applications. It 
enables the handling of various request types, including HTTP, gRPC, TCP, Websocket, Queue Job consuming, and Temporal 
via [spiral/roadrunner-bridge](hhttps://github.com/spiral/roadrunner-bridge) package. The integration unlocks a wide 
range of capabilities for building robust and high-performance applications. 

**Here is a list of features that are available:**

- [HTTP](https://spiral.dev/docs/http-configuration)
- [Static Content](https://spiral.dev/docs/advanced-storage#local-server)
- [Queue](https://spiral.dev/docs/queue-roadrunner) (RabbitMQ, AWS SQS, Beanstalkd, In-Memory, Boltdb, Kafka, NATS)
- [GRPC](https://spiral.dev/docs/grpc-configuration)
- [TCP](https://github.com/spiral/roadrunner-bridge)
- [Key-Value](https://spiral.dev/docs/basics-cache)
- [Websocket](https://spiral.dev/docs/websockets-configuration)
- [Metrics](https://spiral.dev/docs/advanced-prometheus-metrics)
- [OpenTelemetry](https://spiral.dev/docs/advanced-telemetry)
- [Logger](https://spiral.dev/docs/basics-logging#roadrunner-handler)