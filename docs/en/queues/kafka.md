# Jobs — Kafka driver

Kafka driver supported since RoadRunner version `2.11.0`. The Kafka driver has been reworked in `v2023.1.0`.
Apache Kafka is a distributed streaming system used for event stream processing, real-time data pipelines, and
large-scale stream processing.
Originally developed and open-sourced at LinkedIn in 2011, Kafka has quickly evolved from a messaging queue to a
full-fledged event streaming platform.
Now used by 80% of the Fortune 500, Kafka brings numerous benefits to virtually every industry and opens up countless
new use cases, large and small.

Version `2023.2.0` update:

- Added new `SCRAM-SHA-256` and `SCRAM-SHA-512` authentication mechanisms.

## Configuration

```yaml
# Kafka jobs driver
#
# This option is required to use Kafka driver. Addrs can contain any number of addresses separated by comma (127.0.0.1:9092,127.0.0.1:9093,...)
# Kafka jobs driver
#
# This option is required to use Kafka driver,
kafka:

  # Kafka brokers addresses
  #
  # Required to use Kafka driver
  brokers: [ "127.0.0.1:9092", "127.0.0.1:9002" ]

  # Ping to test connection to Kafka
  #
  # Examples: "2s", "5m"
  # Optional, default: "10s"
  ping:
    timeout: "10s"

  # SASL authentication options to use for all connections. Depending on the auth type, plain or aws_msk_plain sections might be removed.
  #
  # Optional, default: empty
  sasl:

    # ----------- 1. PLAIN and SCRAM auth section ---------------

    # Mechanism used for the authentication
    #
    # Required for the section. Might be: 'aws_msk_iam', 'plain', 'SCRAM-SHA-256', 'SCRAM-SHA-512'
    mechanism: plain

    # Username to use for authentication.
    #
    # Required for the plain auth mechanism.
    username: foo

    # Password to use for authentication.
    #
    # Required for the plain auth mechanism.
    password: bar

    # Nonce.
    #
    # Optional for the SHA auth types. Empty by default.
    nonce: "foo"

    # If true, suffixes the "tokenauth=true" extra attribute to the initial authentication message.
    # Set this to true if the user and pass are from a delegation token.
    # Optional for the SHA auth types. Empty by default.
    is_token: false

    # Zid is an optional authorization ID to use in authenticating.
    #
    # Optional, default: empty.
    zid: "foo"

    # -------------- 2. AWS_MSK_IAM auth section ------------------

    # AWS Access key ID.
    #
    # Required
    access_key: foo

    # AWS Secret Access Key.
    #
    #
    secret_key: bar

    # SessionToken, if non-empty, is a session / security token to use for authentication.
    # See the following link for more details:
    #
    # https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html
    session_token: bar

    # UserAgent is the user agent to for the client to use when connecting
    # to Kafka, overriding the default "franz-go/<runtime.Version()>/<hostname>".
    # Setting a UserAgent allows authorizing based on the aws:UserAgent
    # condition key; see the following link for more details:
    #     https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-useragent
    user_agent: baz

jobs:
  num_pollers: 10
  pipeline_size: 100000
  pool:
    num_workers: 10
    max_jobs: 0
    allocate_timeout: 60s
    destroy_timeout: 60s

  pipelines:
    test-local-6:
      # Driver name
      #
      # This option is required
      driver: kafka

      # Driver's configuration
      #
      # Should not be empty
      config:

        # Pipeline priority
        #
        # If the job has priority set to 0, it will inherit the pipeline's priority. Default: 10.
        priority: 1


        # Auto create topic for the consumer/producer
        #
        # Optional, default: false
        auto_create_topics_enable: false

        # Kafka producer options
        #
        # Optional, required only if Push/PushBatch is used.
        producer_options:

          # disable_idempotent disables idempotent produce requests, opting out of
          # Kafka server-side deduplication in the face of reissued requests due to
          # transient network problems.
          # Idempotent production is strictly a win, but does require the IDEMPOTENT_WRITE permission on CLUSTER
          # (pre Kafka 3.0), and not all clients can have that permission.
          #
          # Optional, defaut: false
          disable_idempotent: false

          # required_acks sets the required acks for produced records.
          #
          # Optional, default: AllISRAcks. Possible values: NoAck, LeaderAck, AllISRAck
          required_acks: AllISRAck

          # max_message_bytes upper bounds the size of a record batch, overriding the default 1,000,012 bytes.
          # This mirrors Kafka's max.message.bytes.
          #
          # Optional, default: 1000012
          max_message_bytes: 1000012

          # request_timeout sets how long Kafka broker's are allowed to respond produce requests, overriding the default 10s.
          # If a broker exceeds this duration, it will reply with a request timeout error.
          #
          # Optional, default: 10s. Possible values: 10s, 10m.
          request_timeout: 10s

          # delivery_timeout sets a rough time of how long a record can sit around in a batch before timing out,
          # overriding the unlimited default. If idempotency is enabled (as it is by default), this option is only
          # enforced if it is safe to do so without creating invalid sequence numbers.
          #
          # Optional, default: delivery.timeout.ms Kafka option. Possible values: 10s, 10m.
          delivery_timeout: 100s

          # transaction_timeout sets the allowed for a transaction, overriding the default 40s. It is a good idea to
          # keep this less than a group's session timeout.
          #
          # Optional, default 40s. Possible values: 10s, 10m.
          transaction_timeout: 100

          # compression_codec sets the compression codec to use for producing records.
          #
          # Optional, default is chosen in the order preferred based on broker support. Possible values: gzip, snappy, lz4, zstd.
          compression_codec: gzip

        # Kafka Consumer options. Needed to consume messages from the Kafka cluster.
        #
        # Optional, needed only if `consume` is used.
        consumer_options:

          # topics: adds topics to use for consuming
          #
          # Default: empty (will produce an error), possible to use regexp if `consume_regexp` is set to true.
          topics: [ "foo", "bar", "^[a-zA-Z0-9._-]+$" ]

          # consume_regexp sets the client to parse all topics passed to `topics` as regular expressions.
          # When consuming via regex, every metadata request loads *all* topics, so that all topics can be passed to
          # any regular expressions. Every topic is evaluated only once ever across all regular expressions; either it
          # permanently is known to match, or is permanently known to not match.
          #
          # Optional, default: false.
          consume_regexp: true

          # max_fetch_message_size sets the maximum amount of bytes a broker will try to send during a fetch, overriding the default 50MiB.
          # Note that brokers may not obey this limit if it has records larger than this limit.
          # Also note that this client sends a fetch to each broker concurrently, meaning the client will
          # buffer up to <brokers * max bytes> worth of memory. This corresponds to the Java fetch.max.bytes setting.
          #
          # Optional, default 50000
          max_fetch_message_size: 50000

          # min_fetch_message_size sets the minimum amount of bytes a broker will try to send during a fetch,
          # overriding the default 1 byte. With the default of 1, data is sent as soon as it is available.
          # This corresponds to the Java fetch.min.bytes setting.
          #
          # Optional, default: 1.
          min_fetch_message_size: 1

          # consume_partitions sets partitions to consume from directly and the offsets to start consuming those partitions from.
          # This option is basically a way to explicitly consume from subsets of partitions in topics, or to consume at exact offsets.
          #
          # NOTE: This option is not compatible with group consuming and regex consuming.
          #
          # Optional, default: null
          consume_partitions:

            # Topic for the consume_partitions
            #
            # Required at least one topic.
            foo:

              # Partition for the topic.
              #
              # Required at least one partition.
              0:

                # Partition offset.
                #
                # Required if all options is used. No default, error on empty.
                # Possible values: AtEnd, At, AfterMilli, AtStart, Relative, WithEpoch
                type: AtStart

                # Value for the: At, AfterMilli, Relative and WithEpoch offsets.
                #
                # Optional, default: 0.
                value: 1

          # consumer_offset sets the offset to start consuming from, or if OffsetOutOfRange is seen while fetching,
          # to restart consuming from.
          #
          # Optional, default: AtStart
          consumer_offset:

            # Partition offset.
            #
            # Optional, default: AtStart. Possible values: AtEnd, At, AfterMilli, AtStart, Relative, WithEpoch
            type: AtStart

            # Value for the: At, AfterMilli, Relative and WithEpoch offsets.
            #
            # Optional, default: 0.
            value: 1

        # group_options sets the consumer group for the client to join and consume in.
        # This option is required if using any other group options.
        #
        # Default: empty.
        group_options:

          # group_id sets the group to consume.
          #
          # Required if using group consumer.
          group_id: foo

          # block_rebalance_on_poll switches the client to block rebalances whenever you poll.
          #
          # Optional, default: false.
          block_rebalance_on_poll: true
```
