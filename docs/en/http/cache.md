# HTTP — Cache (RFC7234) middleware

Cache middleware implements http-caching RFC 7234. It's based on the [Souin](https://github.com/darkweak/souin) HTTP
cache library.

Have a look at the [Souin documentation](https://github.com/darkweak/souin) if you need more information.

> **Warning**
> This is a third party plugin and isn't included by default. See the `Building RoadRunner with Cache` section for more
> information.

## Features

* [RFC 7234](https://httpwg.org/specs/rfc7234.html) compliant HTTP Cache.
*

Sets [the `Cache-Status` HTTP Response Header](https://httpwg.org/http-extensions/draft-ietf-httpbis-cache-header.html)

* REST API to purge the cache and list stored resources.
* Builtin support for distributed cache.
* Tag-based invalidation.
* Partial GraphQL caching.
* Configure multiple HTTP verbs to cache (especially for GraphQL).
* Builtin timeout.

## Building RoadRunner with Cache

As it's based on the [Souin](https://github.com/darkweak/souin) HTTP cache library we can use directly the roadrunner
middleware implementation. We have to set the `folder` property because this middleware is located in a subdirectory.

```toml configuration.toml
[github]
[github.token]
token = "YOUR_GH_TOKEN"

[github.plugins]
# Use the Souin third-party middleware.
cache = { ref = "master", owner = "darkweak", repository = "souin", folder = "/plugins/roadrunner" }
# others ...

[log]
level = "debug"
mode = "development"
```

**Available storages**:  
In-memory/Filesystem

- `nutsdb`
- `badger` (default one)

Distributed

- `etcd`
- `olric`

More info about customizing RR with your own plugins: [link](../customization/plugin.md)

## Configuration

You can set each Souin configuration key under the `http.cache` key. There is a configuration example below.

```yaml .rr.yaml
http:
  # Other http sub keys
  cache:
    api:
      basepath: /httpcache_api
      prometheus:
        basepath: /anything-for-prometheus-metrics
      souin: { }
    default_cache:
      allowed_http_verbs:
        - GET
        - POST
        - HEAD
      cdn:
        api_key: XXXX
        dynamic: true
        hostname: XXXX
        network: XXXX
        provider: fastly
        strategy: soft
      headers:
        - Authorization
      regex:
        exclude: '/excluded'
      timeout:
        backend: 5s
        cache: 1ms
      ttl: 5s
      stale: 10s
    log_level: debug
    ykeys:
      The_First_Test:
        headers:
          Content-Type: '.+'
      The_Second_Test:
        url: 'the/second/.+'
    surrogate_keys:
      The_First_Test:
        headers:
          Content-Type: '.+'
      The_Second_Test:
        url: 'the/second/.+'
  middleware:
    - cache
    # Other middlewares
```

**Detailled configuration**

| Key                                | Description                                                           | Value example                                                            |
|:-----------------------------------|:----------------------------------------------------------------------|:-------------------------------------------------------------------------|
| `api`                              | The cache-handler API cache management                                |                                                                          |
| `api.basepath`                     | BasePath for all APIs to avoid  conflicts                             | `/your-non-conflicting-route`<br/><br/>`(default: /souin-api)`           |
| `api.{api}.enable`                 | (DEPRECATED) Enable the API with related routes                       | `true`<br/><br/>`(default: true if you define the api name, false then)` |
| `api.{api}.security`               | (DEPRECATED) Enable the JWT Authentication token  verification        | `true`<br/><br/>`(default: false)`                                       |
| `api.security.secret`              | (DEPRECATED) JWT secret key                                           | `Any_charCanW0rk123`                                                     |
| `api.security.users`               | (DEPRECATED) Array of authorized users with username x password combo | `- username: admin`<br/><br/>`  password: admin`                         |
| `api.souin.security`               | Enable JWT validation to access the resource                                  |  `true`<br/><br/>`(default: false)`                                                                        |
| `cache_keys`                       | Define the key generation rules for each URI matching the key regexp        |                                                                          |
| `cache_keys.{your regexp}`         | Regexp that the URI should match to override the key                  |                                                                          |
| generation                         | `.+\.css`                                                             |                                                                          |
| `default_cache.key.disable_body`   | Disable the body part in the key matching the regexp (GraphQL         |                                                                          |
| context)                           | `true`<br/><br/>`(default: false)`                                    |                                                                          |
| `default_cache.key.disable_host`   | Disable the host part in the key matching the                         |                                                                          |
| regexp                             | `true`<br/><br/>`(default: false)`                                    |                                                                          |
| `default_cache.key.disable_method` | Disable the method part in the key matching the                       |                                                                          |
| regexp                             | `true`<br/><br/>`(default: false)`                                    |                                                                          |
| `cdn`                              | The CDN management, if you use any cdn to proxy your requests         |                                                                          |
| Souin will handle that             |                                                                       |                                                                          |
| `cdn.provider`                     | The provider placed before                                            |                                                                          |
| Souin                              | `akamai`<br/><br/>`fastly`<br/><br/>`souin`                           |                                                                          |
| `cdn.api_key`                      | The api key used to access to the                                     |                                                                          |
| provider                           | `XXXX`                                                                |                                                                          |
| `cdn.dynamic`                      | Enable the dynamic keys returned by your backend                      |                                                                          |
| application                        | `true`<br/><br/>`(default: false)`                                    |                                                                          |
| `cdn.email`                        | The api key used to access to the provider if required, depending     |                                                                          |

the
provider | `XXXX`                                                                                                                    |
| `cdn.hostname`                                    | The hostname if required, depending the
provider | `domain.com`                                                                                                              |
| `cdn.network`                                     | The network if required, depending the
provider | `your_network`                                                                                                            |
| `cdn.strategy`                                    | The strategy to use to purge the cdn cache, soft will keep the
content as a stale
resource | `hard`<br/><br/>`(default: soft)`                                                                                         |
| `cdn.service_id`                                  | The service id if required, depending the
provider | `123456_id`                                                                                                               |
| `cdn.zone_id`                                     | The zone id if required, depending the
provider | `anywhere_zone`                                                                                                           |
| `default_cache.allowed_http_verbs`                | The HTTP verbs to support
cache | `- GET`<br/><br/>`- POST`<br/><br/>`(default: GET, HEAD)`                                                                 |
| `default_cache.badger`                            | Configure the Badger cache storage | |
| `default_cache.badger.path`                       | Configure Badger with a
file | `/anywhere/badger_configuration.json`                                                                                     |
| `default_cache.badger.configuration`              | Configure Badger directly in the Caddyfile or your JSON caddy
configuration | [See the Badger configuration for the options](https://dgraph.io/docs/badger/get-started/)                                |
| `default_cache.etcd`                              | Configure the Etcd cache storage | |
| `default_cache.etcd.configuration`                | Configure Etcd directly in the Caddyfile or your JSON caddy
configuration | [See the Etcd configuration for the options](https://pkg.go.dev/go.etcd.io/etcd/clientv3#Config)                          |
| `default_cache.headers`                           | List of headers to include to the
cache | `- Authorization`<br/><br/>`- Content-Type`<br/><br/>`- X-Additional-Header`                                              |
| `default_cache.key`                               | Override the key generation with the ability to disable unecessary
parts | |
| `default_cache.key.disable_body`                  | Disable the body part in the key (GraphQL
context)                                                                                          | `true`<br/><br/>`(default: false)`                                                                                        |
| `default_cache.key.disable_host`                  | Disable the host part in the
key | `true`<br/><br/>`(default: false)`                                                                                        |
| `default_cache.key.disable_method`                | Disable the method part in the
key | `true`<br/><br/>`(default: false)`                                                                                        |
| `default_cache.etcd`                              | Configure the Etcd cache storage | |
| `default_cache.etcd.configuration`                | Configure Etcd directly in the Caddyfile or your JSON caddy
configuration | [See the Etcd configuration for the options](https://pkg.go.dev/go.etcd.io/etcd/clientv3#Config)                          |
| `default_cache.nuts`                              | Configure the Nuts cache storage | |
| `default_cache.nuts.path`                         | Set the Nuts file path
storage | `/anywhere/nuts/storage`                                                                                                  |
| `default_cache.nuts.configuration`                | Configure Nuts directly in the Caddyfile or your JSON caddy
configuration | [See the Nuts configuration for the options](https://github.com/nutsdb/nutsdb#default-options)                            |
| `default_cache.olric`                             | Configure the Olric cache storage | |
| `default_cache.olric.path`                        | Configure Olric with a
file | `/anywhere/olric_configuration.json`                                                                                      |
| `default_cache.olric.configuration`               | Configure Olric directly in the Caddyfile or your JSON caddy
configuration | [See the Olric configuration for the options](https://github.com/buraksezer/olric/blob/master/cmd/olricd/olricd.yaml/)    |
| `default_cache.port.{web,tls}`                    | The device's local HTTP/TLS port that Souin should be listening
on | Respectively `80`
and `443`                                                                                               |
| `default_cache.regex.exclude`                     | The regex used to prevent paths being
cached | `^[A-z]+.*$`                                                                                                              |
| `default_cache.stale`                             | The stale
duration | `25m`                                                                                                                     |
| `default_cache.timeout`                           | The timeout configuration | |
| `default_cache.timeout.backend`                   | The timeout duration to consider the backend as
unreachable | `10s`                                                                                                                     |
| `default_cache.timeout.cache`                     | The timeout duration to consider the cache provider as
unreachable | `10ms`                                                                                                                    |
| `default_cache.ttl`                               | The TTL
duration | `120s`                                                                                                                    |
| `default_cache.default_cache_control`             | Set the default value of `Cache-Control` response header if not
set by upstream (Souin treats empty `Cache-Control` as `public` if
omitted) | `no-store`                                                                                                                |
| `log_level`                                       | The log
level | `One of DEBUG, INFO, WARN, ERROR, DPANIC, PANIC, FATAL it's case insensitive`                                             |
| `urls.{your url or regex}`                        | List of your custom configuration depending each URL or regex | '
https:\/\/yourdomain.com' |
| `urls.{your url or regex}.ttl`                    | Override the default TTL if
defined | `90s`<br/><br/>`10m`                                                                                                      |
| `urls.{your url or regex}.default_cache_control`  | Override the default default `Cache-Control` if
defined | `public, max-age=86400`                                                                                                   |
| `urls.{your url or regex}.headers`                | Override the default headers if
defined | `- Authorization`<br/><br/>`- 'Content-Type'`                                                                             |
| `surrogate_keys.{key name}.headers`               | Headers that should match to be part of the surrogate key
group | `Authorization: ey.+`<br/><br/>`Content-Type: json`                                                                       |
| `surrogate_keys.{key name}.headers.{header name}` | Header name that should be present a match the regex to be part of
the surrogate key
group | `Content-Type: json`                                                                                                      |
| `surrogate_keys.{key name}.url`                   | Url that should match to be part of the surrogate key
group | `.+`                                                                                                                      |
| `ykeys.{key name}.headers`                        | (DEPRECATED) Headers that should match to be part of the ykey
group | `Authorization: ey.+`<br/><br/>`Content-Type: json`                                                                       |
| `ykeys.{key name}.headers.{header name}`          | (DEPRECATED) Header name that should be present a match the regex
to be part of the ykey
group | `Content-Type: json`                                                                                                      |
| `ykeys.{key name}.url`                            | (DEPRECATED) Url that should match to be part of the ykey
group | `.+`                                                                                                                      |