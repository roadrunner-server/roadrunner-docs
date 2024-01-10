# KV (Key-Value) Plugin — Overview

The RoadRunner KV (Key-Value) plugin is a powerful cache implementation written in Go language. It offers lightning-fast
communication with cache drivers such as:

- [Redis Server](https://redis.io/),
- [Memcached](https://memcached.org/),
- [BoltDB](https://github.com/etcd-io/bbolt) - does not require a separate server,
- [In-memory](memory.md) storage — temporary stores data in RAM.

It is able to handle cache operations more efficiently than the same operations in PHP, leading to faster response times
and improved application performance.

It provides the ability to store arbitrary data inside the RoadRunner between different requests (in case of HTTP
application) or different types of applications. Thus, using [Temporal](https://docs.temporal.io/docs/php/introduction),
for example, you can transfer data inside the [HTTP application](../php/worker.md)and vice versa.

One of the key benefits of using the RoadRunner KV plugin is its RPC interface, which allows for seamless integration
with your existing infrastructure.

![kv-general-info](https://user-images.githubusercontent.com/2461257/128436785-3dadbf0d-13c3-4e0c-859c-4fd9668558c8.png)

## Opentelemetry tracing

All KV drivers support opentelemetry tracing. 
To enable tracing, you need to add [otel](../lab/otel.md) section to your configuration file:

```yaml .rr.yaml
version: "3"

kv:
  example:
    driver: memory
    config: { }

otel:
  resources:
    service_name: "rr_test"
    service_version: "1.0.0"
    service_namespace: "RR-Shop"
    service_instance_id: "UUID"
  insecure: true
  compress: false
  exporter: otlp
  endpoint: 127.0.0.1:4317
```

After that, you can see traces in your [Jaeger](https://www.jaegertracing.io/), [Uptrace](https://uptrace.dev/), [Zipkin](https://zipkin.io/) or any 
other opentelemetry compatible tracing system.


## Configuration

To use the RoadRunner KV plugin, you need to define multiple key-value storages with desired storage drivers in the
configuration file. Each storage must have a `driver` that indicates the type of connection used by those storages. At
the moment, four different types of drivers are available: `boltdb`, `redis`, `memcached`, and `memory`.

> **Note**
> The `memory` and `boltdb` drivers do not require additional binaries and are available immediately, while the rest
> require additional setup. Please see the appropriate documentation for installing [Redis Server](https://redis.io/)
> and/or [Memcached Server](https://memcached.org/).

Here is a simple configuration example:

```yaml .rr.yaml
version: "3"

rpc:
  listen: tcp://127.0.0.1:6001

kv:
  example:
    driver: memory
    config: { }
```

> **Note**
> to interact with the RoadRunner KV plugin, you will need to have the RPC defined in the rpc configuration section. You
> can refer to the documentation page [here](../php/rpc.md) to learn more about the configuration.

## PHP client

The RoadRunner KV plugin comes with a convenient PHP package that simplifies the process of integrating the plugin with
your PHP application and store and request data from storages using RoadRunner PRC.

### Installation

> **Requirements**
> - PHP >= 8.1
> - *ext-protobuf (optional)*

You can install the package via Composer using the following command:

```terminal
composer require spiral/roadrunner-kv
```

## Usage

First, you need to create the RPC connection to the RoadRunner server.

```php
use Spiral\RoadRunner\Environment;
use Spiral\Goridge\RPC\RPC;

// Manual configuration
$rpc = RPC::create('tcp://127.0.0.1:6001');
```

> **Note**
> You can refer to the documentation page [here](../php/rpc.md) to learn more about creating the RPC connection.

To work with storages, you should create the `Spiral\RoadRunner\KeyValue\Factory` object after creating the RPC
connection. It provides a method for selecting the storage.

- `Factory::select(string)`- method receives the name of the storage as the first argument and returns the
  implementation of the [PSR-16](https://www.php-fig.org/psr/psr-16/) `Psr\SimpleCache\CacheInterface` for interacting
  with the key-value RoadRunner storage.

Here is a simple example of using:

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\KeyValue\Factory;

$factory = new Factory($rpc);

$storage = $factory->select('storage-name');

// Expected:
//  An instance of Psr\SimpleCache\CacheInterface interface

$storage->set('key', 'value');

var_dump($storage->get('key'));
// Expected:
//  string(5) "string"
```

The RoadRunner KV API provides several additional methods, such as `getTtl(string)` and `getMultipleTtl(string)`, which
allow you to get information about the expiration of an item stored in a key-value storage.

> Please note that the `memcached` driver
> [**does not support**](https://github.com/memcached/memcached/issues/239)
> these methods.

```php
$ttl = $factory
    ->select('memory')
    ->getTtl('key');
    
// Expected:
//  - An instance of \DateTimeInterface if "key" expiration time is available
//  - Or null otherwise

$ttl = $factory
    ->select('memcached')
    ->getTtl('key');
    
// Expected:
//  Spiral\RoadRunner\KeyValue\Exception\KeyValueException: Storage "memcached"
//  does not support kv.TTL RPC method execution. Please use another driver for
//  the storage if you require this functionality.
```

### Value Serialization

To save and receive data from the key-value store, the data serialization mechanism is used. This way you can store and
receive arbitrary serializable objects.

```php
$storage->set('test', (object)['key' => 'value']);

$item = $storage->set('test');
// Expected:
//  object(StdClass)#399 (1) {
//    ["key"] => string(5) "value"
//  }
```

If you need to specify your custom serializer, you can do so by specifying it in the key-value factory constructor as a
second argument, or by using the `Factory::withSerializer(SerializerInterface): self` method. This will allow you to use
your own serialization mechanism and store more complex objects in the key-value store.

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\KeyValue\Factory;

$storage = (new Factory($rpc))
    ->withSerializer(new CustomSerializer())
    ->select('storage');
```

If you require a specific serializer for a particular value stored in the key-value storage, you can use
the `withSerializer()` method. This allows you to use a custom serializer for that particular value while still using
the default serializer for other values.

```php
// Using default serializer
$storage->set('key', 'value');

// Using custom serializer
$storage
    ->withSerializer(new CustomSerializer())
    ->set('key', 'value');
```

#### Igbinary Value Serialization

The serialization mechanism in PHP is not always efficient, which can impact the performance of your application. To
increase the speed of serialization and deserialization, it is recommended to use
the [igbinary extension](https://github.com/igbinary/igbinary).

:::: tabs

::: tab Linux and MacOS
In a Linux and MacOS environment, it may be installed with a simple command:

```bash
pecl install igbinary
```

:::

::: tab Windows
For the Windows OS, you can download it from the
[PECL website](https://windows.php.net/downloads/pecl/releases/igbinary/).
:::

::::

> **Note**
> More detailed installation instructions are [available here](https://github.com/igbinary/igbinary#installing).

Here is an example of using the `igbinary` serializer:

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\KeyValue\Factory;
use Spiral\RoadRunner\KeyValue\Serializer\IgbinarySerializer;

$storage = (new Factory($rpc)
    ->withSerializer(new IgbinarySerializer())
    ->select('storage');
```

#### End-to-End Value Encryption

Some data may contain sensitive information, such as personal data of the user. In these cases, it is recommended to use
data encryption.

> **Note**
> To use encryption, you need to install the [Sodium extension](https://www.php.net/manual/en/book.sodium.php).

Next, you should have an encryption key generated using
[sodium_crypto_box_keypair()](https://www.php.net/manual/en/function.sodium-crypto-box-keypair.php) function. You can do
this using the following command:

```bash
php -r "echo sodium_crypto_box_keypair();" > keypair.key
```

> **Warning**
> Do not store security keys in a control versioning system (like GIT)!

After generating the keypair, you can use it to encrypt and decrypt the data.

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\KeyValue\Factory;
use Spiral\RoadRunner\KeyValue\Serializer\SodiumSerializer;
use Spiral\RoadRunner\KeyValue\Serializer\DefaultSerializer;

$storage = new Factory($rpc);
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

## API

### Protobuf API

To make it easy to use the KV proto API in PHP, we provide
a [GitHub repository](https://github.com/roadrunner-php/roadrunner-api-dto), that contains all the generated PHP DTO
classes proto files, making it easy to work with these files in your PHP application.

- [API](https://github.com/roadrunner-server/api/blob/master/kv/v1/kv.proto)

### RPC API

RoadRunner provides an RPC API, which allows you to manage key-value in your applications using remote procedure calls.
The RPC API provides a set of methods that map to the available methods of the `Spiral\RoadRunner\KeyValue\Cache` class
in PHP.

#### Has

Checks for the presence of one or more keys in the specified storage.

```go
func (r *rpc) Has(in *kvv1.Request, out *kvv1.Response) error {}
```

#### Set

Sets one or more key-value pairs in the specified storage.

```go
func (r *rpc) Set(in *kvv1.Request, _ *kvv1.Response) error {}
```

#### MGet

Gets the values of one or more keys from the specified storage.

```go
func (r *rpc) MGet(in *kvv1.Request, out *kvv1.Response) error {}
```

#### MExpire

Sets the expiration time for one or more keys in the specified storage.

```go
func (r *rpc) MExpire(in *kvv1.Request, _ *kvv1.Response) error {}
```

#### TTL

Gets the expiration time of a single key in the specified storage.

```go
func (r *rpc) TTL(in *kvv1.Request, out *kvv1.Response) error {}
```

#### Delete

Deletes one or more keys from the specified storage.

```go
func (r *rpc) Delete(in *kvv1.Request, _ *kvv1.Response) error {}
```

#### Clear

Clears all keys from the specified storage.

```go
func (r *rpc) `Clear(in *kvv1.Request, _ *kvv1.Response) error {}
```

### Example

To use the RPC API in PHP, you can create an RPC connection to the RoadRunner server and use the `call()` method to
perform the desired operation. For example, to call the `MGet` method, you can use the following code:

```php
use Spiral\Goridge\RPC\RPC;
use Spiral\Goridge\RPC\Codec\ProtobufCodec;
use RoadRunner\KV\DTO\V1\{Request, Response};

$response = RPC::create('tcp://127.0.0.1:6001')
    ->withServicePrefix('kv')
    ->withCodec(new ProtobufCodec())
    ->call('MGet', new Request([ ... ]), Response::class);
```
