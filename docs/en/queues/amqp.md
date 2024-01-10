# Jobs — AMQP Driver

Strictly speaking, AMQP (and 0.9.1 version used) is a protocol, not a full-fledged driver, so you can use any servers
that support this protocol (on your own, only rabbitmq was tested), such as:

- [RabbitMQ](https://www.rabbitmq.com/),
- [Apache Qpid](http://qpid.apache.org/)
- [Apache ActiveMQ](http://activemq.apache.org/).

However, it is recommended to use RabbitMQ as the main implementation, and reliable performance with other
implementations is not guaranteed.

To install and configure the RabbitMQ, use the
corresponding [documentation page](https://www.rabbitmq.com/download.html).

After that, you should configure the connection to the server in the `amqp` section. This configuration section
contains exactly one `addr` key with a [connection DSN](https://www.rabbitmq.com/uri-spec.html). The `TLS` configuration sits in the `amqp.tls` section and consists of the following options:

- `key`: path to a key file.
- `cert`: path to a certificate file.
- `root_ca`: CA certificate path, used with the `client_auth_type`.
- `client_auth_type`: also known as `mTLS`. Possible values are: `request_client_cert`, `require_any_client_cert`, `verify_client_cert_if_given`, `require_and_verify_client_cert`, `no_client_certs`.

You should also configure `rabbitMQ` with `TLS` support: [link](https://www.rabbitmq.com/ssl.html).

```yaml .rr.yaml
amqp:
  addr: amqp://guest:guest@127.0.0.1:5672

  # AMQPS TLS configuration
  #
  # This section is optional
  tls:
    # Path to the key file
    #
    # This option is required
    key: ""

    # Path to the certificate
    #
    # This option is required
    cert: ""

    # Path to the CA certificate, defines the set of root certificate authorities that servers use if required to verify a client certificate. Used with the `client_auth_type` option.
    #
    # This option is optional
    root_ca: ""

    # Client auth type (mTLS, peer verification).
    #
    # This option is optional. Default value: no_client_certs. Possible values: request_client_cert, require_any_client_cert, verify_client_cert_if_given, require_and_verify_client_cert, no_client_certs
    client_auth_type: no_client_certs
```

Upon establishing a connection to the server, you can create a new queue that utilizes this connection and encompasses
the queue settings, including those specific to AMQP).

## Configuration

```yaml .rr.yaml
version: "3"

amqp:
  addr: amqp://guest:guest@127.0.0.1:5672 

  # AMQPS TLS configuration
  #
  # This section is optional
  tls:
    # Path to the key file
    #
    # This option is required
    key: ""

    # Path to the certificate
    #
    # This option is required
    cert: ""

    # Path to the CA certificate, defines the set of root certificate authorities that servers use if required to verify a client certificate. Used with the `client_auth_type` option.
    #
    # This option is optional
    root_ca: ""

    # Client auth type (mTLS, peer verification).
    #
    # This option is optional. Default value: no_client_certs. Possible values: request_client_cert, require_any_client_cert, verify_client_cert_if_given, require_and_verify_client_cert, no_client_certs
    client_auth_type: no_client_certs

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
        exchange_auto_delete: false

        # Auto-delete (queue that has had at least one consumer is deleted when last consumer unsubscribes) (rabbitmq option: https://www.rabbitmq.com/queues.html#properties)
        #
        # Default: false
        queue_auto_delete: false

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
        # Default: false
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

## Configuration options

**Here is a detailed description of each of the amqp-specific options:**

### Priority

`priority`- job priority. A lower value corresponds to a higher priority. For instance, consider two pipelines: `pipe1`
with a priority of `1` and `pipe10` with a priority of `10`. Workers will only take jobs from `pipe10` if all the jobs
from `pipe1` have been processed.

### Prefetch

`prefetch` - rabbitMQ QoS prefetch. See also ["prefetch-size"](https://www.rabbitmq.com/amqp-0-9-1-reference.html#basic.qos.prefetch-size). Note that if you use a large number of workers and a small `prefetch` number, some of the workers may not be loaded with messages (jobs) due to the blocking nature of the prefetch. This would result in poor RoadRunner performance and waste of resources. 

### Queue

`queue` - required, AMQP internal (inside the driver) queue name.

### Consume all

`consume_all` - by default, RoadRunner only supports `Jobs` structures from the queue. Enable this option by setting it
to true if you wish to consume raw payloads as well.

### Exchange

`exchange` - required, rabbitMQ exchange name.

> **Note**
> See also [AMQP model](https://www.rabbitmq.com/tutorials/amqp-concepts.html#amqp-model) documentation section.

### Exchange type

`exchange_type` - rabbitMQ exchange type. May be one of `direct`, `topics`,`headers` or `fanout`.

### Routing key

`routing_key` - queue's routing key. Required to push the `Job`.

### Exclusive

`exclusive` - applied to the queue, exclusive queues cannot be redeclared. If set to true, and you attempt to declare
the same pipeline twice, it will result in an error.

### Multiple ack

`multiple_ack` - this delivery, along with all prior unacknowledged deliveries on the same channel, will be
acknowledged. This feature is beneficial for batch processing of deliveries and is applicable only for `Ack`, not for
`Nack`.

### Requeue on fail

`requeue_on_fail` - requeue on Nack (by RabbitMQ).

> **Note**
> Read more about Nack in RabbitMQ official docs: https://www.rabbitmq.com/confirms.html#consumer-nacks-requeue

### Queue headers

`queue_headers` - used to pass arguments to the `Queue` create method, such as `x-queue-mode: lazy`

### Durable

`durable` - create a durable queue. 

Default: `false`

### Delete queue on stop

`delete_queue_on_stop` - delete the queue when the pipeline is stopped. 

Default: `false`

### Redial timeout

`redial_timeout` - Redial timeout (in seconds). How long to try to reconnect to the AMQP server.

### Exchange durable

`exchange_durable` - Durable
exchange ([rabbitmq option](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges)). 

Default: `false`

### Exchange auto delete

`exchange_auto_delete` - Auto-delete (exchange is deleted when last queue is unbound from it): [link](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges). 

Default: `false`

### Queue auto delete

`queue_auto_delete` - Auto-delete (queue that has had at least one consumer is deleted when last consumer
unsubscribes): [link](https://www.rabbitmq.com/queues.html#properties). 

Default: `false`

### Consumer id

`consumer_id` - string that is unique and scoped for all consumers on this channel.
