# KV (Key-Value) Plugin

The Key-Value plugin provides the ability to store arbitrary data inside the
RoadRunner between different requests (in case of HTTP application) or different
types of applications. Thus, using [Temporal](https://docs.temporal.io/docs/php/introduction),
for example, you can transfer data inside the [HTTP application](../php/worker.md)
and vice versa.

As a permanent source of data, the RoadRunner allows you to use popular solutions,
such as [Redis Server](https://redis.io/) or [Memcached](https://memcached.org/),
but in addition it provides others that do not require a separate server, such
as [BoltDB](https://github.com/etcd-io/bbolt), and also allows you to replace
permanent storage with temporary that stores data in RAM.

![kv-general-info](https://user-images.githubusercontent.com/2461257/128436785-3dadbf0d-13c3-4e0c-859c-4fd9668558c8.png)

## Installation

> **Requirements**
> - PHP >= 8.1
> - RoadRunner >= 2.3
> - *ext-protobuf (optional)*

To get access from the PHP code, you should put the corresponding dependency
using [the Composer](https://getcomposer.org/).

```sh
$ composer require spiral/roadrunner-kv
```

## Configuration

After installing all the required dependencies, you need to configure this
plugin. To enable it, add the `kv` section to your configuration:

```yaml
version: "3"

rpc:
  listen: tcp://127.0.0.1:6001

kv:
  example:
    driver: memory
    config: {}
```

Please note that to interact with the KV, you will also need the RPC defined
in the `rpc` configuration section. You can read more about the configuration and
methods of creating the RPC connection on the [documentation page here](../php/rpc.md).

This configuration initializes this plugin with one storage named "`example`".
In addition, each storage must have a `driver` that indicates the type of
connection used by those storages. In total, at the moment, 4 different types of
drivers are available with their own characteristics and additional settings:
`boltdb`, `redis`, `memcached`, and `memory`.

The `memory` and `boltdb` drivers do not require additional binaries and are
available immediately, while the rest require additional setup. Please see
the appropriate documentation for installing [Redis Server](https://redis.io/)
and/or [Memcached Server](https://memcached.org/).

## Usage

First, you need to create the RPC connection to the RoadRunner server. You can
specify an address with a connection by hands or use automatic detection if
you run the php code as a [RoadRunner Worker](/php/worker.md).

```php
use Spiral\RoadRunner\Environment;
use Spiral\Goridge\RPC\RPC;

// Manual configuration
$rpc = RPC::create('tcp://127.0.0.1:6001');

// Autodetection
$env = Environment::fromGlobals();
$rpc = RPC::create($env->getRPCAddress());
```

After creating the RPC connection, you should create the
`Spiral\RoadRunner\KeyValue\Factory` object for working with storages of KV
RoadRunner plugin.

The factory object provides two methods for working with the plugin.

- Method `Factory::isAvailable(): bool` returns boolean `true` value if the
  plugin is available and `false` otherwise. Note, that this method will always return an `Exception` because it was removed from the RR RPC since `v2.6.2`, [issue](https://github.com/roadrunner-server/roadrunner/issues/901).  In the releases after `v2.6.2` you can safely remove calls to that method.

- Method `Factory::select(string): CacheInterface` receives the name of the
  storage as the first argument and returns the implementation of the
  [PSR-16](https://www.php-fig.org/psr/psr-16/) `Psr\SimpleCache\CacheInterface`
  for interact with the key-value RoadRunner storage.

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\KeyValue\Factory;

$factory = new Factory(RPC::create('tcp://127.0.0.1:6001'));

if (!$factory->isAvailable()) {
    throw new \LogicException('The "kv" RoadRunner plugin not available');
}

$storage = $factory->select('storage-name');
// Expected:
//  An instance of Psr\SimpleCache\CacheInterface interface

$storage->set('key', 'value');

echo $storage->get('key');
// Expected:
//  string(5) "string"
```

> The `clear()` method available since [RoadRunner v2.3.1](https://github.com/roadrunner-server/roadrunner/releases/tag/v2.3.1).

Apart from this, RoadRunner Key-Value API provides several additional methods:
You can use `getTtl(string): ?\DateTimeInterface` and
`getMultipleTtl(string): iterable<\DateTimeInterface|null>` methods to get
information about the expiration of an item stored in a key-value storage.

> Please note that the `memcached` driver
> [**does not support**](https://github.com/memcached/memcached/issues/239)
> these methods.

```php
$ttl = $factory->select('memory')
    ->getTtl('key');
// Expected:
//  - An instance of \DateTimeInterface if "key" expiration time is available
//  - Or null otherwise

$ttl = $factory->select('memcached')
    ->getTtl('key');
// Expected:
//  Spiral\RoadRunner\KeyValue\Exception\KeyValueException: Storage "memcached"
//  does not support kv.TTL RPC method execution. Please use another driver for
//  the storage if you require this functionality.
```

### Value Serialization

To save and receive data from the key-value store, the data serialization
mechanism is used. This way you can store and receive arbitrary serializable
objects.

```php
$storage->set('test', (object)['key' => 'value']);

$item = $storage->set('test');
// Expected:
//  object(StdClass)#399 (1) {
//    ["key"] => string(5) "value"
//  }
```

To specify your custom serializer, you will need to specify it in the key-value
factory constructor as a second argument, or use the
`Factory::withSerializer(SerializerInterface): self` method.

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\KeyValue\Factory;

$connection = RPC::create('tcp://127.0.0.1:6001');

$storage = (new Factory($connection))
    ->withSerializer(new CustomSerializer())
    ->select('storage');
```

In the case that you need a specific serializer for a specific value from the
storage, then you can use a similar method `withSerializer()` for a specific
storage.

```php
// Using default serializer
$storage->set('key', 'value');

// Using custom serializer
$storage
    ->withSerializer(new CustomSerializer())
    ->set('key', 'value');
```


#### Igbinary Value Serialization

As you know, the serialization mechanism in PHP is not always productive. To
increase the speed of work, it is recommended to use the
[ignbinary extension](https://github.com/igbinary/igbinary).

- For the Windows OS, you can download it from the
  [PECL website](https://windows.php.net/downloads/pecl/releases/igbinary/).

- In a Linux and MacOS environment, it may be installed with a simple command:
```sh
$ pecl install igbinary
```

More detailed installation instructions are [available here](https://github.com/igbinary/igbinary#installing).

After installing the extension, you just need to install the desired igbinary
serializer in the factory instance.

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\KeyValue\Factory;
use Spiral\RoadRunner\KeyValue\Serializer\IgbinarySerializer;

$storage = (new Factory(RPC::create('tcp://127.0.0.1:6001')))
    ->withSerializer(new IgbinarySerializer())
    ->select('storage');
//
// Now this $storage is using igbinary serializer.
//
```

#### End-to-End Value Encryption

Some data may contain sensitive information, such as personal data of the user.
In these cases, it is recommended to use data encryption.

To use encryption, you need to install the
[Sodium extension](https://www.php.net/manual/en/book.sodium.php).

Next, you should have an encryption key generated using
[sodium_crypto_box_keypair()](https://www.php.net/manual/en/function.sodium-crypto-box-keypair.php)
function. You can do this using the following command:
```sh
$ php -r "echo sodium_crypto_box_keypair();" > keypair.key
```

> Do not store security keys in a control versioning systems (like GIT)!

After generating the keypair, you can use it to encrypt and decrypt the data.

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\KeyValue\Factory;
use Spiral\RoadRunner\KeyValue\Serializer\SodiumSerializer;
use Spiral\RoadRunner\KeyValue\Serializer\DefaultSerializer;

$storage = new Factory(RPC::create('tcp://127.0.0.1:6001'));
    ->select('storage');

// Encrypted serializer
$key = file_get_contents(__DIR__ . '/path/to/keypair.key');
$encrypted = new SodiumSerializer($storage->getSerializer(), $key);

// Storing public data
$storage->set('user.login', 'test');

// Storing private data
$storage->withSerializer($encrypted)
    ->set('user.email', 'test@example.com');
```

## RPC Interface

All communication between PHP and GO made by the RPC calls with protobuf payloads.
You can find versioned proto-payloads here: [Proto](https://github.com/roadrunner-server/api/blob/master/kv/v1/kv.proto).

- `Has(in *kvv1.Request, out *kvv1.Response)` - The arguments: the first argument
  is a `Request` , which declares a `storage` and an array of `Items` ; the second
  argument is a `Response`, it will contain `Items` with keys which are present in
  the provided via `Request` storage. Item value and timeout are not present in
  the response.  The error returned if the request fails.

- `Set(in *kvv1.Request, _ *kvv1.Response)` - The arguments: the first argument
  is a `Request` with the `Items` to set; return value isn't used and present here
  only because GO's RPC calling convention. The error returned if request fails.

- `MGet(in *kvv1.Request, out *kvv1.Response)` - The arguments: the first
  argument is a `Request` with `Items` which should contain only keys (server
  doesn't check other fields); the second argument is `Response` with the `Items`.
  Every item will have `key` and `value` set, but without timeout (See: `TTL`).
  The error returned if request fails.

- `MExpire(in *kvv1.Request, _ *kvv1.Response)` - The arguments: the first
  argument is a `Request` with `Items` which should contain keys and timeouts set;
  return value isn't used and present here only because GO's RPC calling convention.
  The error returned if request fails.

- `TTL(in *kvv1.Request, out *kvv1.Response)` - The arguments: the first argument
  is a `Request` with `Items` which should contain keys; return value will contain
  keys with their timeouts. The error returned if request fails.

- `Delete(in *kvv1.Request, _ *kvv1.Response)` - The arguments: the first
  argument is a `Request` with `Items` which should contain keys to delete; return
  value isn't used and present here only because GO's RPC calling convention.
  The error returned if request fails.

- `Clear(in *kvv1.Request, _ *kvv1.Response)` - The arguments: the first
  argument is a `Request` with `storage` which should contain the storage to be
  cleaned up; return value isn't used and present here only because GO's RPC
  calling convention. The error returned if request fails.

From the PHP point of view, such requests (`MGet` for example) are as follows:

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\Goridge\RPC\Codec\ProtobufCodec;
use Spiral\RoadRunner\KeyValue\DTO\V1\{Request, Response};

$response = RPC::create('tcp://127.0.0.1:6001')
    ->withServicePrefix('kv')
    ->withCodec(new ProtobufCodec())
    ->call('MGet', new Request([ ... ]), Response::class);
```
