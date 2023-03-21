# Jobs (Queue) plugin

Starting with RoadRunner >= 2.4, a queuing system (aka "jobs") is available.
This plugin allows you to move arbitrary "heavy" code into separate tasks to
execute them asynchronously in an external worker, which will be referred to
as "consumer" in this documentation.

The RoadRunner PHP library provides both API implementations: The client one,
which allows you to dispatch tasks, and the server one, which provides the
consumer who processes the tasks.

![queue](https://user-images.githubusercontent.com/2461257/128100380-2d4df71a-c86e-4d5d-a58e-a3d503349200.png)

## Installation

> **Requirements**
> - PHP >= 8.1
> - RoadRunner >= 2.4
> - *ext-protobuf (optional, but would increase the performance a lot)*

To get access from the PHP code, you should put the corresponding dependency
using [the Composer](https://getcomposer.org/).

```sh
$ composer require spiral/roadrunner-jobs
```

## Configuration

After installing all the required dependencies, you need to configure this
plugin. To enable it add `jobs` section to your configuration.

For example, in this way, you can configure both the client and server parts to
work with RabbitMQ.

```yaml
version: "3"
#
# RPC is required for tasks dispatching (client)
#
rpc:
  listen: tcp://127.0.0.1:6001

#
# This section configures the task consumer (server)
#
server:
  command: php consumer.php
  relay: pipes

#
# In this section, the jobs themselves are configured
#
jobs:
  consume: [ "test" ]   # List of RoadRunner queues that can be processed by 
  # the consumer specified in the "server" section.
  pipelines:
    test: # RoadRunner queue identifier
      driver: memory    # - Queue driver name
      config:
        priority: 10
        prefetch: 10
```

- The `rpc` section is responsible for client settings. It is at this address
  that we will connect, *dispatching tasks* to the queue.

- The `server` section is responsible for configuring the server. Previously, we
  have already met with its description when setting up the [PHP Worker](/php/worker.md).

- And finally, the `jobs` section is responsible for the work of the queues
  themselves. It contains information on how the RoadRunner should work with
  connections to drivers, what can be handled by the consumer, and other
  queue-specific settings.

### Common Configuration

Let's now focus on the common settings of the queue server. In full, it may
look like this:

```yaml
version: "3"

jobs:
  num_pollers: 64
  timeout: 60
  pipeline_size: 100000
  pool:
    num_workers: 10
    allocate_timeout: 60s
    destroy_timeout: 60s
  consume: [ "queue-name" ]
  pipelines:
    queue-name:
      driver: # "[DRIVER_NAME]"
      config: # NEW in 2.7
      # And driver-specific configuration below...
```

Above is a complete list of all possible common Jobs settings. Let's now figure
out what they are responsible for.

- `num_pollers` - The number of threads that concurrently read from the priority
  queue and send payloads to the workers. There is no optimal number, it's
  heavily dependent on the PHP worker's performance. For example, "echo workers"
  may process over 300k jobs per second within 64 pollers (on 32 core CPU).

- `timeout` - The internal timeouts via golang context (in seconds). For
  example, if the connection was interrupted or your push in the middle of the
  redial state with 10 minutes timeout (but our timeout is 1 min for example),
  or queue is full. If the timeout exceeds, your call will be rejected with an
  error. Default: 60 (seconds).

- `pipeline_size` - The "binary heaps" priority queue (PQ) settings. Priority
  queue stores jobs inside according to its' priorities. Priority might be set
  for the job or inherited by the pipeline. If worker performance is poor, PQ
  will accumulate jobs until `pipeline_size` will be reached. After that, PQ
  will be blocked until workers process all the jobs inside.

  Blocked PQ means, that you can push the job into the driver, but RoadRunner
  will not read that job until PQ will be empty. If RoadRunner will be killed
  with jobs inside the PQ, they won't be lost, because jobs are deleted from the
  drivers' queue only after Ack.

- `pool` - All settings in this section are similar to the worker pool settings
  described on the [configuration page](https://roadrunner.dev/docs/intro-config).

- `consume` - Contains an array of the names of all queues specified in the
  `"pipelines"` section, which should be processed by the concierge specified in
  the global `"server"` section (see the [PHP worker's settings](/php/worker.md)).

- `pipelines` - This section contains a list of all queues declared in the
  RoadRunner. The key is a unique *queue identifier*, and the value is an object
  from the settings specific to each driver (we will talk about it later).