# What is it?

This is RoadRunner. RoadRunner is a High-performance PHP application server, process manager written in Go and powered with plugins. It runs
your application in the form of workers (processes).

## RoadRunner

RoadRunner manages a set of PHP processes (called workers) and passes the incoming requests from different plugins
via the [goridge](https://github.com/roadrunner-server/goridge) protocol to them.

![Base Diagram](https://user-images.githubusercontent.com/796136/65347341-79dd8600-dbe7-11e9-9621-1c5f2ef929e6.png)

The data can be received from the HTTP request, AWS Lambda, Queue, KV, gRPC, Temporal plugins. 

## PHP

RoadRunner keeps PHP workers alive between incoming requests. This means that you can completely eliminate bootload time
(such as framework initialization) and significantly speed up a heavy application.

![Base Diagram](https://user-images.githubusercontent.com/796136/65348057-00df2e00-dbe9-11e9-9173-f0bd4269c101.png)

Since a worker is located in the memory, all open resources will remain open for the next request. 
Using [goridge](https://github.com/roadrunner-server/goridge)
RPC built-in functionality, you can quickly offload some complex computations to the application server. 
For example, schedule a background PHP Job.