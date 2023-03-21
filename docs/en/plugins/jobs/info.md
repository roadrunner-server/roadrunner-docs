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

## PHP Client (Producer)

Now that we have configured the server, we can start writing our first code for
sending the task to the queue. But before doing this, we need to connect to our
server. And to do this, it is enough to create a `Jobs` instance.

```php
// Server Connection
$jobs = new Spiral\RoadRunner\Jobs\Jobs();
```

Please note that in this case we have not specified any connection settings. And
this is really not required if this code is executed in a RoadRunner environment.
However, in the case that a connection is required to be established
from a third-party application (for example, a CLI command), then the settings
must be specified explicitly.

```php
$jobs = new Spiral\RoadRunner\Jobs\Jobs(
    // Expects RPC connection
    Spiral\Goridge\RPC\RPC::create('tcp://127.0.0.1:6001')
);
```

After we have established the connection, we should check the server
availability and in this case the API availability for the jobs. This can be
done using the appropriate `isAvailable()` method.

> **Note**, that this method will always return an `Exception` because it was removed
> from the RR RPC since `v2.6.2`, [issue](https://github.com/roadrunner-server/roadrunner/issues/901).
> In the releases after `v2.6.2` you can safely remove calls to that method.

When the connection is created, and the availability of the functionality is checked, we can connect to the queue we
need using `connect()` method.


```php
$jobs = new Spiral\RoadRunner\Jobs\Jobs();

// Be careful, isAvailable method doesn't work after v2.6.2
if (!$jobs->isAvailable()) {
    throw new LogicException('The server does not support "jobs" functionality =(');
}

$queue = $jobs->connect('queue-name');
```

### Task Creation

Before submitting a task to the queue, you should create this task. To create a
task, it is enough to call the corresponding `create()` method.

```php
$task = $queue->create(SendEmailTask::class);
// Expected:
// object(Spiral\RoadRunner\Jobs\Task\PreparedTaskInterface)
```

> Note that the name of the task does not have to be a class. Here we are using
> `SendEmailTask` just for convenience.

Also, this method takes an additional second argument with additional data to
complete this task.

```php
$task = $queue->create(
  SendEmailTask::class, 
  payload: ['email' => 'dev@null.pipe']
);
```

You can also use this task as a basis for creating several others.

```php
$task = $queue->create(SendEmailTask::class);

$first = $task->withValue('john.doe@example.com');
$second = $task->withValue('john.snow@the-wall.north');
```

In addition, the method takes an additional third argument with `Spiral\RoadRunner\Jobs\OptionsInterface`
where you can pass object with predefined options.

```php
$options = new \Spiral\RoadRunner\Jobs\Options(autoAck: true);

$task = $queue->create(
  SendEmailTask::class, 
  options: $options->withDelay(10)
);
```

You can also redefine options for created task.

```php
$options = new \Spiral\RoadRunner\Jobs\Options(autoAck: true);

$task = $queue->create(SendEmailTask::class);

$task = $task->withOptions($options);
```

### Task creation for Kafka driver

Please note, a queue with Kafka driver requires a task with specified `topic`. In this case
you have to use `Spiral\RoadRunner\Jobs\KafkaOptionsInterface`, because it has all required
methods for working with Kafka driver. Connect to queue using `Spiral\RoadRunner\Jobs\KafkaOptionsInterface`.
To redefine this options for a particular message, simply pass another `Spiral\RoadRunner\Jobs\KafkaOptionsInterface`
implementation as a second parameter of the `create` method.

```php
$options = new \Spiral\RoadRunner\Jobs\KafkaOptions(topic: 'topic_name');

$queue = $jobs->connect('queue_name', $options);

$task = $queue->create(
  SendEmailTask::class,
  options: $options->withPartition(10)->withOffset(1)->withMetadata('foo=bar')
);
```

As you noticed we use everywhere interfaces for setting options for the task, and a developer
has the ability create his own implementation of the interfaces.

### Task Dispatching

And to send tasks to the queue, we can use different methods:
`dispatch()` and `dispatchMany()`. The difference between these two
implementations is that the first one sends a task to the queue, returning a
dispatched task object, while the second one dispatches multiple tasks,
returning an array. Moreover, the second method provides one-time delivery of
all tasks in the array, as opposed to sending each task separately.

```php
$a = $queue->create(SendEmailTask::class, ['email' => 'john.doe@example.com']);
$b = $queue->create(SendEmailTask::class, ['email' => 'john.snow@the-wall.north']);

foreach ([$a, $b] as $task) {
    $result = $queue->dispatch($task);
    // Expected:
    // object(Spiral\RoadRunner\Jobs\Task\QueuedTaskInterface)
}

// Using a batching send
$result = $queue->dispatchMany($a, $b);
// Expected:
// array(2) {
//    object(Spiral\RoadRunner\Jobs\Task\QueuedTaskInterface),
//    object(Spiral\RoadRunner\Jobs\Task\QueuedTaskInterface)
// }
```

### Task Immediately Dispatching

In the case that you do not want to create a new task and then immediately
dispatch it, you can simplify the work by using the `push` method. However, this
functionality has a number of limitations. In case of creating a new task:
- You can flexibly configure additional task capabilities using a convenient
  fluent interface.
- You can prepare a common task for several others and use it as a basis to
  create several alternative tasks.
- You can create several different tasks and collect them into one collection
  and send them to the queue at once (using the so-called batching).

In the case of immediate dispatch, you will have access to only the basic
features: The `push()` method accepts one required argument with the
name of the task and two optional arguments containing additional data for the
task being performed and additional sending options (for example, a delay).
Moreover, this method is designed to send only one task.

```php
use Spiral\RoadRunner\Jobs\Options;

$payload = ['email' => $email, 'message' => $message];

$task = $queue->push(SendEmailTask::class, $payload, new Options(
    delay: 60 // in seconds
));
```

### Task Payload

As you can see, each task, in addition to the name, can contain additional data
(payload) specific to a certain type of task. You yourself can determine what
data should be transferred to the task and no special requirements are imposed
on them, except for the main ones: Since this task is then sent to the queue,
they must be serializable.

> The default serializer used in jobs allows you to pass anonymous functions
> as well.

In case to add additional data, you can use the optional second argument
provided by the `create()` and `push()` methods, or you can use the fluent
interface to supplement or modify the task data. Everything is quite simple
here; you can add data using the `withValue()` method, or delete them using the
`withoutValue()` method.

The first argument of the `withValue()` method passes a payload value as the
required first argument. If you also need to specify a key for it, just pass it
as an optional second argument.

```php
$task = $queue->create(CreateBackup::class)
    ->withValue('/var/www')
    ->withValue(42, 'answer')
    ->withValue('/dev/null', 'output');
    
// An example like this will be completely equivalent to if we passed
// all this data at one time
$task = $queue->create(CreateBackup::class, [
    '/var/www',
    'answer' => 42,
    'output' => '/dev/null'
]);

// On the other hand, we don't need an "answer"...
$task = $task->withoutValue('answer');
```

### Task Headers

In addition to the data itself, we can send additional metadata that is not
related to the payload of the task, that is, headers. In them, we can pass
any additional information, for example: Encoding of messages, their format,
the server's IP address, the user's token or session id, etc.

Headers can only contain string values and are not serialized in any way during
transmission, so be careful when specifying them.

In the case to add a new header to the task, you can use methods
[similar to PSR-7](https://www.php-fig.org/psr/psr-7/). That is:
- `withHeader(string, iterable<string>|string): self` - Return an instance with
  the provided value replacing the specified header.
- `withAddedHeader(string, iterable<string>|string): self` - Return an instance
  with the specified header appended with the given value.
- `withoutHeader(string): self` - Return an instance without the specified header.

```php
$task = $queue->create(RestartServer::class)
    ->withValue('addr', '127.0.0.1')
    ->withAddedHeader('access-token', 'IDDQD');

$queue->dispatch($task);
```

### Task Delayed Dispatching

If you want to specify that a job should not be immediately available for
processing by a jobs worker, you can use the delayed job option.
For example, let's specify that a job shouldn't be available for processing
until 42 minutes after it has been dispatched:

```php
$task = $queue->create(SendEmailTask::class)
    ->withDelay(42 * 60); // 42 min * 60 sec
```

## Consumer Usage

You probably already noticed that when setting up a jobs consumer,
the `"server"` configuration section is used in which a PHP file-handler is defined.
Exactly the same one we used earlier to write a [HTTP Worker](../../php/worker.md).
Does this mean that if we want to use the Jobs Worker, then we can no longer
use the HTTP Worker? No it does not!

During the launch of the RoadRunner, it spawns several workers defined in the
`"server"` config section (by default, the number of workers is equal to the
number of CPU cores). At the same time, during the spawn of the workers, it
transmits in advance to each of them information about the *mode* in which this
worker will be used. The information about the *mode* itself is contained in the
environment variable `RR_ENV` and for the HTTP worker the value will correspond
to the `"http"`, and for the Jobs worker the value of `"jobs"` will be stored
there.

![queue-mode](https://user-images.githubusercontent.com/2461257/128106755-cb0d3cb7-3f98-433e-a1c7-1ed92839376a.png)

There are several ways to check the operating mode from the code:
- By getting the value of the env variable.
- Or using the appropriate API method (from the `spiral/roadrunner-worker` package).

The second choice may be more preferable in cases where you need to change the
RoadRunner's mode, for example, in tests.

```php
use Spiral\RoadRunner\Environment;
use Spiral\RoadRunner\Environment\Mode;

// 1. Using global env variable
$isJobsMode = $_SERVER['RR_MODE'] === 'jobs';

// 2. Using RoadRunner's API
$env = Environment::fromGlobals();

$isJobsMode = $env->getMode() === Mode::MODE_JOBS;
```

After we are convinced of the specialization of the worker, we can write the
corresponding code for processing tasks. To get information about the available
task in the worker, use the
`$consumer->waitTask(): ReceivedTaskInterface` method.

```php
use Spiral\RoadRunner\Jobs\Consumer;
use Spiral\RoadRunner\Jobs\Task\ReceivedTaskInterface;


$consumer = new Consumer();

/** @var Spiral\RoadRunner\Jobs\Task\ReceivedTaskInterface $task */
while ($task = $consumer->waitTask()) {
    var_dump($task);
}
```

After you receive the task from the queue, you can start processing it in
accordance with the requirements. Don't worry about how much memory or time this
execution takes - the RoadRunner takes over the tasks of managing and
distributing tasks among the workers.

After you have processed the incoming task, you can execute the
`complete(): void` method. After that, you tell the RoadRunner that you are
ready to handle the next task.

```php
$consumer = new Spiral\RoadRunner\Jobs\Consumer();

while ($task = $consumer->waitTask()) {

    //
    // Task handler code
    //
    
    $task->complete();
}
```

We got acquainted with the possibilities of receiving and processing tasks, but
we do not yet know what the received task is. Let's see what data it contains.

### Task Failing

In some cases, an error may occur during task processing. In this case, you
should use the `fail()` method, informing the RoadRunner about it. The method
takes two arguments. The first argument is required and expects any string or
string-like (instance of Stringable, for example any exception) value with an
error message. The second is optional and tells the server to restart this task.

```php
$consumer = new Spiral\RoadRunner\Jobs\Consumer();
$shouldBeRestarted = false;

while ($task = $consumer->waitTask()) {
    try {
        //
        // Do something...
        //
        $task->complete();
    } catch (\Throwable $e) {
        $task->fail($e, $shouldBeRestarted);
    }
}
```

In the case that the next time you restart the task, you should update the
headers, you can use the appropriate method by adding or changing the headers
of the received task.

```php
$task
    ->withHeader('attempts', (int)$task->getHeaderLine('attempts') - 1)
    ->withHeader('retry-delay', (int)$task->getHeaderLine('retry-delay') * 2)
    ->fail('Something went wrong', requeue: true)
;
```

In addition, you can re-specify the task execution delay. For example, in the
code above, you may have noticed the use of a custom header `"retry-delay"`, the
value of which doubled after each restart, so this value can be used to specify
the delay in the next task execution.

```php
$task
    ->withDelay((int)$task->getHeaderLine('retry-delay'))
    ->fail('Something went wrong', true)
;
```

### Received Task ID

Each task in the queue has a **unique** identifier. This allows you to
unambiguously identify the task among all existing tasks in all queues, no
matter what name it was received from.

In addition, it is worth paying attention to the fact that the identifier is not
a sequential number that increases indefinitely. It means that there is still a
chance of an identifier collision, but it is about 1/2.71 quintillion. Even if
you send 1 billion tasks per second, it will take you about 85 years for an ID
collision to occur.

```php
echo $task->getId(); 
// Expected Result
// string(36) "88ca6810-eab9-473d-a8fd-4b4ae457b7dc"
```

In the case that you want to store this identifier in the database, it is
recommended to use a binary representation (16 bytes long if your DB requires
blob sizes).

```php
$binary = hex2bin(str_replace('-', '', $task->getId()));
// Expected Result
// string(16) b"ˆÊh\x10ê¹G=¨ýKJäW·Ü"
```

### Received Task Queue

Since a worker can process several different queues at once, you may need to
somehow determine from which queue the task came. To get the name of the queue,
use the `getQueue(): string` method.

```php
echo $task->getQueue();
// Expected
// string(13) "example-queue"
```

For example, you can select different task handlers based on different types of
queues.

```php
// This is just an example of a handler
$handler = $container->get(match($task->getQueue()) {
    'emails'  => 'email-handler',
    'billing' => 'billing-handler',
    default   => throw new InvalidArgumentException('Unprocessable queue [' . $task->getQueue() . ']')
});

$handler->process($task);
```

### Task auto acknowledge

RoadRunner version `v2.10.0+` supports an auto acknowledge task option. You might use this option to acknowledge a task right after RR receive it from the queue.
You can use this option for the non-important tasks which can fail or break the worker.

To use this option you may update the Options:

```php
// Create with default values
$options = new Options(
     Options::DEFAULT_DELAY,
     Options::DEFAULT_PRIORITY,
     Options::DEFAULT_AUTO_ACK, // false by default
);
```

Or manage that manually per every `Task`:

```php
use Spiral\RoadRunner\Jobs\Queue\MemoryCreateInfo;
use Spiral\RoadRunner\Jobs\Options;
use Spiral\RoadRunner\Jobs\Jobs;

$options = new Options();

// Jobs service
$jobs = new Jobs(RPC::create('tcp://127.0.0.1:6001'));

// Select "test" queue from jobs
$queue = $jobs->connect('test');

// or create a new queue
$queue = $jobs->create(new MemoryCreateInfo('local'));

// Set default auto ack for all tasks
$queue = $queue->withDefaultOptions(
     $options->withAutoAck(true)
);

// Create a new task with custom auto ack
$task = $queue->push('task_name', ['foo' => 'bar'], (new Options())->withAutoAck(false));

// or change auto ack for created task
$task = $queue->create('task_name', ['foo' => 'bar'])->withAutoAck(false);
$queue->dispatch($task);
```

### Received Task Name

The task name is some identifier associated with a specific type of task. For
example, it may contain the name of the task class so that in the future we can
create an object of this task by passing the required data there. To get the
name of the task, use the `getName(): string` method.

```php
echo $task->getName();
// Expected
// string(21) "App\\Queue\\Task\\EmailTask"
```

Thus, we can implement the creation of a specific task with certain data for
this task.

```php
$class = $task->getName();

if (!class_exists($class)) {
    throw new InvalidArgumentException("Unprocessable task [$class]");
}

$handler->process($class::fromTask($task));
```

### Received Task Payload

Each task contains a set of arbitrary user data to be processed within the task.
To obtain this data, you can use one of the available methods:

**getValue**

Method `getValue()` returns a specific payload value by key or `null` if no
value was passed. If you want to specify any other default value (for those
cases when the payload with the identifier was not passed), then use the second
argument, passing your own default value there.

```php
if ($task->getName() !== SendEmailTask::class) {
    throw new InvalidArgumentException('Does not look like a mail task');
}

echo $task->getValue('email');              // "john.doe@example.com"
echo $task->getValue('username', 'Guest');  // "John"
```

**hasValue**

To check the existence of any value in the payload, use the `hasValue()` method.
This method will return `true` if the value for the payload was passed and `false`
otherwise.

```php
if (!$task->hasValue('email')) {
    throw new InvalidArgumentException('The "email" value is required for this task');
}

$email->sendTo($task->getValue('email'));
```

**getPayload**

Also you can get all data at once in `array(string|int $key => mixed $value)`
format using the `getPayload` method. This method may be useful to you in cases
of transferring all data to the DTO.

```php
$class = $task->getName();
$arguments = $task->getPayload();

$dto = new $class(...$arguments);
```

You should pay attention that an array can contain both `int` and  `string`
keys, so you should take care of their correct pass to the constructor
yourself. For example, the code above will work completely correctly only in the
case of PHP >= 8.1. And in the case of earlier versions of the language, you
should use the [reflection functionality](https://www.php.net/manual/ru/reflectionclass.newinstanceargs.php),
or pass the payload in some other way.

Since the handler process is not the one that put this task in the queue, then
if you send any object to the queue, it will be serialized and then automatically
unpacked in the handler. The default serializer suitable for most cases, so you
can even pass `Closure` instances. However, in the case of any specific data
types, you should manage their packing and unpacking yourself, either by
replacing the serializer completely, or for a separate value. In this case, do
not forget to specify this both on the client and consumer side.

### Received Task Headers

In the case that you need to get any additional information that is not related
to the task, then for this you should use the functionality of headers.

For example, headers can convey information about the serializer, encoding, or
other metadata.

```php
$message = $task->getValue('message');
$encoding = $task->getHeaderLine('encoding');

if (strtolower($encoding) !== 'utf-8') {
    $message = iconv($encoding, 'utf-8', $message);
}
```

The interface for receiving headers is completely similar to
[PSR-7](https://www.php-fig.org/psr/psr-7/), so methods are available to you:
- `getHeaders(): array<string, array<string, string>>` - Retrieves all task
  header values.
- `hasHeader(string): bool` - Checks if a header exists by the given name.
- `getHeader(string): array<string, string>` - Retrieves a message header value
  by the given name.
- `getHeaderLine(string): string` - Retrieves a comma-separated string of the
  values for a single header by the given name.

We got acquainted with the data and capabilities that we have in the consumer.
Let's now get down to the basics - sending these messages.

## Advanced Functionality

In addition to the main functionality of queues for sending and processing in
API has additional functionality that is not directly related to these tasks.
After we have examined the main functionality, it's time to disassemble the
advanced features.

### Creating A New Queue

In the very [first chapter](https://roadrunner.dev/docs/plugins-jobs/2.x/en#configuration), we got acquainted
with the queue settings and drivers for them. In approximately the same way, we
can do almost the same thing with the help of the PHP code using `create()`
method through `Jobs` instance.

To create a new queue, the following types of DTO are available to you:

- `Spiral\RoadRunner\Jobs\Queue\AMQPCreateInfo` for AMQP queues.
- `Spiral\RoadRunner\Jobs\Queue\BeanstalkCreateInfo` for Beanstalk queues.
- `Spiral\RoadRunner\Jobs\Queue\MemoryCreateInfo` for in-memory queues.
- `Spiral\RoadRunner\Jobs\Queue\SQSCreateInfo` for SQS queues.
- `Spiral\RoadRunner\Jobs\Queue\KafkaCreateInfo` for Kafka queues.
- `Spiral\RoadRunner\Jobs\Queue\BoltdbCreateInfo` for Boltdb queues.

Such a DTO with the appropriate settings should be passed to the `create()`
method to create the corresponding queue:

```php
use Spiral\RoadRunner\Jobs\Jobs;
use Spiral\RoadRunner\Jobs\Queue\MemoryCreateInfo;

$jobs = new Jobs();

//
// Create a new "example" in-memory queue
//
$queue = $jobs->create(new MemoryCreateInfo(
    name: 'example',
    priority: 42,
    prefetch: 10,
));
```

### Getting A List Of Queues

In that case, to get a list of all available queues, you just need to use the
standard functionality of the `foreach` operator. Each element of this collection
will correspond to a specific queue registered in the RoadRunner. And to simply
get the number of all available queues, you can pass a `Job` object to the
`count()` function.

```php
$jobs = new Spiral\RoadRunner\Jobs\Jobs();

foreach ($jobs as $queue) {
    var_dump($queue->getName()); 
    // Expects name of the queue
}

$count = count($jobs);
// Expects the number of a queues
```

### Pausing A Queue

In addition to the ability to create new queues, there may be times when a queue
needs to be suspended for processing. Such cases can arise, for example, in the
case of deploying a new application, when the processing of tasks should be
suspended during the deployment of new application code.

In this case, the code will be pretty simple. It is enough to call the `pause()`
method, passing the names of the queues there. In order to start the work of
queues further (unpause), you need to call a similar `resume()` method.

```php
$jobs = new Spiral\RoadRunner\Jobs\Jobs();

// Pause "emails", "billing" and "backups" queues.
$jobs->pause('emails', 'billing', 'backups');

// Resuming only "emails" and "billing".
$jobs->resume('emails', 'billing');
```

## RPC Interface

All communication between PHP and GO made by the RPC calls with protobuf payloads.
You can find versioned proto-payloads here: [Proto](https://github.com/roadrunner-server/roadrunner/blob/e9713a1d08a93e2be70c889c600ed89f54822b54/proto/jobs/v1beta).

- `Push(in *jobsv1.PushRequest, out *jobsv1.Empty) error` - The
  arguments: the first argument is a `PushRequest`, which contains one field
  of the `Job` being sent to the queue; the second argument is `Empty`, which
  means that the function does not return a result (returns nothing). The error
  returned if the request fails.

- `PushBatch(in *jobsv1.PushBatchRequest, out *jobsv1.Empty) error` -
  The arguments: the first argument is a `PushBatchRequest`, which contains one
  repeated (list) field of the `Job` being sent to the queue; the second
  argument is `Empty`, which means that the function does not return a result.
  The error returned if the request fails.

- `Pause(in *jobsv1.Pipelines, out *jobsv1.Empty) error` - The arguments:
  the first argument is a `Pipelines`, which contains one repeated (list)
  field with the `string` names of the queues to be paused; the second
  argument is `Empty`, which means that the function does not return a result.
  The error returned if the request fails.

- `Resume(in *jobsv1.Pipelines, out *jobsv1.Empty) error` - The
  arguments: the first argument is a `Pipelines`, which contains one
  repeated (list) field with the `string` names of the queues to be resumed; the
  second argument is `Empty`, which means that the function does not return a
  result. The error returned if the request fails.

- `List(in *jobsv1.Empty, out *jobsv1.Pipelines) error` - The
  arguments: the first argument is an `Empty`, meaning that the function does
  not accept anything (from the point of view of the PHP API, an empty string
  should be passed); the second argument is `Pipelines`, which contains one
  repeated (list) field with the `string` names of the all available queues.
  The error returned if the request fails.

- `Declare(in *jobsv1.DeclareRequest, out *jobsv1.Empty) error` - The
  arguments: the first argument is an `DeclareRequest`, which contains one
  `map<string, string>` pipeline field of queue configuration; the second
  argument is `Empty`, which means that the function does not return a result.
  The error returned if the request fails.

- `Stat(in *jobsv1beta.Empty, out *jobsv1beta.Stats) error` - The arguments:
  the first argument is an `Empty`, meaning that the function does not accept
  anything (from the point of view of the PHP API, an empty string should be
  passed); the second argument is `Stats`, which contains one repeated (list)
  field named `Stats` of type `Stat`. The error returned if the request fails.


From the PHP point of view, such requests (`List` for example) are as follows:
```php
use Spiral\Goridge\RPC\RPC;
use Spiral\Goridge\RPC\Codec\ProtobufCodec;
use Spiral\RoadRunner\Jobs\DTO\V1\Maintenance;

$response = RPC::create('tcp://127.0.0.1:6001')
    ->withServicePrefix('jobs')
    ->withCodec(new ProtobufCodec())
    ->call('List', '', Maintenance::class);
```