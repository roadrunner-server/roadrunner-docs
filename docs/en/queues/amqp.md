# AMQP Driver

Strictly speaking, AMQP (and 0.9.1 version used) is a protocol, not a full-fledged driver, so you can use
any servers that support this protocol (on your own, only rabbitmq was tested) , such as:
[RabbitMQ](https://www.rabbitmq.com/), [Apache Qpid](http://qpid.apache.org/) or
[Apache ActiveMQ](http://activemq.apache.org/). However, it is recommended to
use RabbitMQ as the main implementation, and reliable performance with other
implementations is not guaranteed.

To install and configure the RabbitMQ, use the corresponding
[documentation page](https://www.rabbitmq.com/download.html). After that, you
should configure the connection to the server in the "`amqp`" section. This
configuration section contains exactly one `addr` key with a
[connection DSN](https://www.rabbitmq.com/uri-spec.html).

```yaml
amqp:
  addr: amqp://guest:guest@localhost:5672
```

Upon establishing a connection to the server, you can create a new queue that utilizes this connection and encompasses the queue settings, including those specific to AMQP):

```yaml
version: "3"

amqp:
  addr: amqp://guest:guest@localhost:5672

jobs:
  pipelines:
    # User defined name of the queue.
    example:
      # Driver name
      #
      # This option is required.
      driver: amqp

      # Driver's configuration
      #
      # Should not be empty
      config:

        # QoS - prefetch.
        #
        # Default: 10
        prefetch: 10

        # Pipeline priority
        #
        # If the job has priority set to 0, it will inherit the pipeline's priority. Default: 10.
        priority: 1

        # Consume any payload type (not only Jobs structured)
        #
        # Default: false
        consume_all: false
        
        # Redial timeout (in seconds). How long to try to reconnect to the AMQP server.
        #
        # Default: 60
        redial_timeout: 60

        # Durable queue
        #
        # Default: false
        durable: false

        # Durable exchange (rabbitmq option: https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges)
        #
        # Default: false
        exchange_durable: false

        # Auto-delete (exchange is deleted when last queue is unbound from it): https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges
        #
        # Default: false
        exchange_auto_deleted: false

        # Auto-delete (queue that has had at least one consumer is deleted when last consumer unsubscribes) (rabbitmq option: https://www.rabbitmq.com/queues.html#properties)
        #
        # Default: false
        queue_auto_deleted: false

        # Delete queue when stopping the pipeline
        #
        # Default: false
        delete_queue_on_stop: false

        # Queue name
        #
        # Default: default
        queue: test-1-queue

        # Exchange name
        #
        # Default: amqp.default
        exchange: default

        # Exchange type
        #
        # Default: direct.
        exchange_type: direct

        # Routing key for the queue
        #
        # Default: empty.
        routing_key: test

        # Declare a queue exclusive at the exchange
        #
        # Default: false
        exclusive: false

        # When multiple is true, this delivery and all prior unacknowledged deliveries
        # on the same channel will be acknowledged.  This is useful for batch processing
        # of deliveries
        #
        # Default:false
        multiple_ack: false

        # The consumer_id is identified by a string that is unique and scoped for all consumers on this channel.
        #
        # Default: "roadrunner" + uuid.
        consumer_id: "roadrunner-uuid"

        # Use rabbitmq mechanism to requeue the job on fail
        #
        # Default: false
        requeue_on_fail: false
        
        # Queue headers (new in 2.12.2)
        #
        # Default: null
        queue_headers:
          x-queue-mode: lazy
```

Below is a more detailed description of each of the amqp-specific options:
- `priority`: job priority. A lower value corresponds to a higher priority. For instance, consider two pipelines: "pipe1" with a priority of 1 and "pipe10" with a priority of 10. Workers will only take jobs from "pipe10" if all the jobs from "pipe1" have been processed.
- `prefetch`: rabbitMQ QoS prefetch. See also ["prefetch-size"](https://www.rabbitmq.com/amqp-0-9-1-reference.html)
- `queue`: required, AMQP internal (inside the driver) queue name.
- `consume_all`: by default, RoadRunner only supports `Jobs` structures from the queue. Enable this option by setting it to true if you wish to consume raw payloads as well.
- `exchange`: required, rabbitMQ exchange name. See also ["AMQP model"](https://www.rabbitmq.com/tutorials/amqp-concepts.html#amqp-model) documentation section.
- `exchange_type`: rabbitMQ exchange type. May be one of `direct`, `topics`,`headers` or `fanout`.
- `routing_key`: queue's routing key. Required to push the `Job`.
- `exclusive`: applied to the queue, exclusive queues cannot be redeclared. If set to true, and you attempt to declare the same pipeline twice, it will result in an error.
- `multiple_ack`:  this delivery, along with all prior unacknowledged deliveries on the same channel, will be acknowledged. This feature is beneficial for batch processing of deliveries and is applicable only for Ack, not for Nack.
- `requeue_on_fail`: requeue on Nack (by RabbitMQ). Docs: https://www.rabbitmq.com/confirms.html#consumer-nacks-requeue
- `queue_headers`: used to pass arguments to the `Queue` create method, such as `x-queue-mode: lazy`

**NEW in 2.7:**

- `durable`: create a durable queue. Default: false
- `delete_queue_on_stop`: delete the queue when the pipeline is stopped. Default: false

**NEW in 2.12:**

- `redial_timeout`: Redial timeout (in seconds). How long to try to reconnect to the AMQP server.
- `exchange_durable`: Durable exchange ([rabbitmq option](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges)). Default: false
- `exchange_auto_deleted`: Auto-delete (exchange is deleted when last queue is unbound from it): [link](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges). Default: false
- `queue_auto_deleted`: Auto-delete (queue that has had at least one consumer is deleted when last consumer unsubscribes): [link](https://www.rabbitmq.com/queues.html#properties). Default: false

**NEW in 2.12.2:**

`queue_headers`: used to pass arguments to the `Queue` create method, such as `x-queue-mode: lazy`

**NEW in 2023.1.0:**

`consumer_id`: string that is unique and scoped for all consumers on this channel.
