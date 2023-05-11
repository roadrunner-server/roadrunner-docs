# Jobs — Beanstalk Driver

Beanstalk is a simple and fast general purpose work queue. To install Beanstalk, you can use
the [local queue server](https://github.com/beanstalkd/beanstalkd) or run the server
inside [AWS Elastic](https://aws.amazon.com/elasticbeanstalk/). You can choose any option that is convenient for you.

Setting up the server is similar to setting up AMQP and requires specifying the connection in the `beanstalk` section of
your RoadRunner configuration file.

```yaml .rr.yaml
beanstalk:
  addr: tcp://127.0.0.1:11300
```

## Configuration

```yaml .rr.yaml
version: "3"

beanstalk:
  # Optional section.
  # Default: tcp://127.0.0.1:11300
  addr: tcp://127.0.0.1:11300

  # Optional section.
  # Default: 30s
  timeout: 10s

jobs:
  pipelines:
    # User defined name of the queue.
    beanstalk-pipeline-name:
      # Required section.
      # Should be "beanstalk" for the Beanstalk driver.
      driver: beanstalk

      config: # NEW in 2.7

        # Optional section.
        # Default: 10
        priority: 10

        # Consume any payload type (not only Jobs structured)
        # Default: false
        consume_all: false

        # Optional section.
        # Default: 1
        tube_priority: 1

        # Optional section.
        # Default: default
        tube: default

        # Optional section.
        # Default: 5s
        reserve_timeout: 5s
```

## Configuration options

**Here is a detailed description of each of the beanstalk-specific options:**

### Priority

`priority` - Similar to the same option in other drivers. This is queue default priority for each task pushed into this
queue if the priority value for these tasks was not explicitly set. Lower value - higher priority.

### Tube priority

`tube_priority` - The value for specifying the priority within Beanstalk is the internal priority of the server. The
value should not exceed `int32` size.

### Tube

`tube` - The name of the inner "tube" specific to the Beanstalk driver.

### Consume all

`consume_all` - By default, RR supports only `Jobs` structures from the queue. Set this option to true if you want to
also consume the raw payloads.