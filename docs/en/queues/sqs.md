# Jobs — SQS Driver

[Amazon SQS Simple Queue Service](https://aws.amazon.com/sqs/) is an alternative
queue server also developed by Amazon and is also part of the AWS
service infrastructure. If you prefer to use the "cloud" option, you can use the
[prebuilt documentation](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configuring.html)
for its installation.

In addition to the ability to use this queue server within AWS, you can also use the
can also use the local installation of this system on your own servers. If you prefer
this option, you can use the [softwaremill's implementation](https://github.com/softwaremill/elasticmq)
of the Amazon SQS server.

After you have created the SQS server, you need to specify the following
connection settings in the `sqs` configuration settings. Unlike AMQP and Beanstalk,
SQS requires more values to set up a connection and will be different from what we are used to.
we're used to.

## Configuration

```yaml .rr.yaml
sqs:
  # Required AccessKey ID.
  # Default: empty
  key: access-key

  # Required secret access key.
  # Default: empty
  secret: api-secret

  # Required AWS region.
  # Default: empty
  region: us-west-1

  # Required AWS session token.
  # Default: empty
  session_token: test

  # Required AWS SQS endpoint to connect.
  # Default: http://127.0.0.1:9324
  endpoint: http://127.0.0.1:9324
```

> **Note**
> Please note that although each of the sections contains default values, it is marked as "required". This means that in
> almost all cases they are required to be specified in order to correctly configure the driver.

After you have configured the connection - you should configure the queue that will use this connection:

> **Note**
> You may also skip the whole `sqs` configuration section (global, not the pipeline) to use the AWS IAM credentials if
> the RR is inside the EC2 machine. RR will try to detect env automatically by making a http request to
> the `http://169.254.169.254/latest/dynamic/instance-identity/` as
> pointer [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/identify_ec2_instances.html)

```yaml .rr.yaml
version: "3"

sqs:
# SQS connection configuration...

jobs:
  pipelines:
    test-sqs-pipeline:
      # Required section.
      # Should be "sqs" for the Amazon SQS driver.
      driver: sqs

      config:
        # Optional section.
        # Default: 10
        prefetch: 10

        # Consume any payload type (not only Jobs structured)
        # Default: false
        consume_all: false

        # Get queue URL only
        # Default: false
        skip_queue_declaration: false

        # Optional section.
        # Default: 0
        visibility_timeout: 0

        # Optional section.
        # Default: 0
        wait_time_seconds: 0

        # Optional section.
        # Default: default
        queue: default

        # Message group ID: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessage.html#SQS-SendMessage-request-MessageGroupId
        # Default: empty, should be set if FIFO queue is used
        message_group_id: "test"

        # Optional section.
        # Default: empty
        attributes:
          DelaySeconds: 42
          # etc... see https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SetQueueAttributes.html

        # Optional section.
        # Default: empty
        tags:
          test: "tag"
```

## Configuration options

**Here is a detailed description of each of the SQS-specific options:**

### Prefetch

`prefetch` - Number of jobs to prefetch from the SQS until ACK/NACK.
Default: `10`.

### Visibility timeout

`visibility_timeout` - The duration (in seconds) that the received messages are hidden from subsequent retrieve requests
after being retrieved by a `ReceiveMessage` request. Max value is `43200` seconds (12 hours). Default: `0`.

### Wait time seconds

`wait_time_seconds` - The duration (in seconds) for which the call waits for a message to arrive in the queue before
returning. If a message is available, the call returns sooner than WaitTimeSeconds. If no messages are available and the
wait time expires, the call returns successfully with an empty list of messages.

Default: `5`.

### Queue

`queue` - SQS internal queue name. Can contain alphanumeric characters, hyphens (`-`), and underscores (`_`).

Default value is `default` string.

### Message Group ID

`message_group_id` - Message group ID is required for FIFO queues. Messages that belong to the same message group are processed in a FIFO manner.
More info: [link](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessage.html#SQS-SendMessage-request-MessageGroupId)

### Skip queue declaration

`skip_queue_declaration` - By default, RR tries to declare the queue by default and then gets the queue URL. Set this
option to `true` if the user already declared the queue to only get its URL.

### Consume all

`consume_all` - By default, RR supports only `Jobs` structures from the queue. Set this option to true if you want to
also consume the raw payloads.

### Attributes

`attributes` - List of
the [AWS SQS attributes](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SetQueueAttributes.html).

```yaml
attributes:
  DelaySeconds: 0
  MaximumMessageSize: 262144
  MessageRetentionPeriod: 345600
  ReceiveMessageWaitTimeSeconds: 0
  VisibilityTimeout: 30
```

### Tags

`tags` - Tags don't have any semantic meaning. Amazon SQS interprets tags as character.

> **Note**
> This functionality is rarely used and slows down the work of
> queues: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-queue-tags.html
