# Jobs — NATS Driver

NATS driver supported in RR since `v2.5.0` and includes only NATS JetStream support.

## Configuration

```yaml .rr.yaml
version: "3"

nats:
  addr: "demo.nats.io"

jobs:
  num_pollers: 10
  pipeline_size: 100000
  pool:
    num_workers: 10
    max_jobs: 0
    allocate_timeout: 60s
    destroy_timeout: 60s

  pipelines:
    test-1:
      driver: nats
      config:
        # Pipeline priority
        # If the job has priority set to 0, it will inherit the pipeline's priority. Default: 10.
        priority: 2

        # NATS prefetch
        # Messages to read into the channel
        prefetch: 100

        # Consume any payload type (not only Jobs structured)
        # Default: false
        consume_all: false

        # NATS subject
        # Default: default
        subject: default

        # NATS stream
        # Default: default-stream
        stream: foo

        # The consumer will only start receiving messages that were created after the consumer was created
        # Default: false (deliver all messages from the stream beginning)
        deliver_new: true

        # Consumer rate-limiter in bytes https://docs.nats.io/jetstream/concepts/consumers#ratelimit
        # Default: 1000
        rate_limit: 100

        # Delete the stream when after pipeline was stopped
        # Default: false
        delete_stream_on_stop: false

        # Delete message from the stream after successful acknowledge
        # Default: false
        delete_after_ack: false
```

## Configuration options

**Here is a detailed description of each of the nats-specific options:**

### Subject

`subject` - nats [subject](https://docs.nats.io/nats-concepts/subjects).

### Stream

`stream` - stream name.

### Deliver new

`deliver_new` - the consumer will only start receiving messages that were created after the consumer was created.

### Rate limit

`rate_limit` - NATS rate [limiter](https://docs.nats.io/jetstream/concepts/consumers#ratelimit).

### Delete stream on stop

`delete_stream_on_stop` - delete the whole stream when pipeline stopped.

### Delete after ack

`delete_after_ack` - delete message after it successfully acknowledged.

### Consume all

`consume_all` - By default, RR supports only `Jobs` structures from the queue. Set this option to true if you want to
also consume the raw payloads.