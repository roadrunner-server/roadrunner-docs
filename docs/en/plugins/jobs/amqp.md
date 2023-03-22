### AMQP Driver

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

After creating a connection to the server, you can create a new queue that will
use this connection and which will contain the queue settings (including
amqp-specific):

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
- `priority` - Queue default priority for each task pushed into this queue
  if the priority value for these tasks was not explicitly set.
  Lower value - higher priority.
  For example, we have 2 pipelines "pipe1" with priority 1 and "pipe10" with priority 10. Jobs from "pipe10" will be taken by workers only if all the jobs from "pipe1" are handled.

- `prefetch` - The client can request that messages be sent in advance so that
  when the client finishes processing a message, the following message is
  already held locally, rather than needing to be sent down the channel.
  Prefetching gives a performance improvement. This field specifies the prefetch
  window size in octets. See also ["prefetch-size"](https://www.rabbitmq.com/amqp-0-9-1-reference.html)
  in AMQP QoS documentation reference.

- `queue`: required, AMQP internal (inside the driver) queue name.

- `consume_all` - By default, RR supports only `Jobs` structures from the queue. Set this option to true if you want to also consume the raw payloads.

- `exchange` - The name of AMQP exchange to which tasks are sent. Exchange
  distributes the tasks to one or more queues. It routes tasks to the queue
  based on the created bindings between it and the queue. See also
  ["AMQP model"](https://www.rabbitmq.com/tutorials/amqp-concepts.html#amqp-model)
  documentation section.

- `exchange_type` - The type of task delivery. May be one of `direct`, `topics`,
  `headers` or `fanout`.
    - `direct` - Used when a task needs to be delivered to specific queues. The
      task is published to an exchanger with a specific routing key and goes to
      all queues that are associated with this exchanger with a similar routing
      key.
    - `topics` - Similarly, `direct` exchange enables selective routing by
      comparing the routing key. But, in this case, the key is set using a
      template, like: `user.*.messages`.
    - `fanout` - All tasks are delivered to all queues even if a routing key is
      specified in the task.
    - `headers` - Routes tasks to related queues based on a comparison of the
      (key, value) pairs of the headers property of the binding and the similar
      property of the message.

    - `routing_key` - Queue's routing key.

    - `exclusive` - Exclusive queues can't be redeclared. If set to true and
      you'll try to declare the same pipeline twice, that will lead to an error.

    - `multiple_ack` - This delivery and all prior unacknowledged deliveries on
      the same channel will be acknowledged. This is useful for batch processing
      of deliveries. Applicable only for the Ack, not for the Nack.

    - `requeue_on_fail` - Requeue on Nack (by RabbitMQ). Docs: https://www.rabbitmq.com/confirms.html#consumer-nacks-requeue
    - `queue_headers` - is used to pass arguments to the `Queue` create method, such as `x-queue-mode: lazy`

**NEW in 2.7:**

- `durable`: create a durable queue. Default: false
- `delete_queue_on_stop`: delete the queue when the pipeline is stopped. Default: false

**NEW in 2.12:**

- `redial_timeout`: Redial timeout (in seconds). How long to try to reconnect to the AMQP server.
- `exchange_durable`: Durable exchange ([rabbitmq option](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges)). Default: false
- `exchange_auto_deleted`: Auto-delete (exchange is deleted when last queue is unbound from it): [link](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges). Default: false
- `queue_auto_deleted`: Auto-delete (queue that has had at least one consumer is deleted when last consumer unsubscribes): [link](https://www.rabbitmq.com/queues.html#properties). Default: false

**NEW in 2.12.2:**

`queue_headers` - is used to pass arguments to the `Queue` create method, such as `x-queue-mode: lazy`

**NEW in 2023.1.0:**

`queue_headers` - is used to pass arguments to the `Queue` create method, such as `x-queue-mode: lazy`
