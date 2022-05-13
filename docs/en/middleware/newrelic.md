### NewRelic HTTP middleware

- ✏️ **[BETA]** Support for the New Relic observability platform. Sample of the client library might be
  found [here](https://github.com/arku31/roadrunner-newrelic). (Thanks @arku31)
  New Relic middleware is a part of the HTTP plugin, thus configuration should be inside it:

```yaml
http:
  address: 127.0.0.1:15389
  middleware: [ "new_relic" ] <------- NEW
  new_relic: <---------- NEW
    app_name: "app"
    license_key: "key"
  pool:
    num_workers: 10
    allocate_timeout: 60s
    destroy_timeout: 60s
```

License key and application name could be set via environment variables: (leave `app_name` and `license_key` empty)

- license_key: `NEW_RELIC_LICENSE_KEY`.
- app_name: `NEW_RELIC_APP_NAME`.

To set the New Relic attributes, the PHP worker should send headers values withing the `rr_newrelic` header key.
Attributes should be separated by the `:`, for example `foo:bar`, where `foo` is a key and `bar` is a value. New Relic
attributes sent from the worker will not appear in the HTTP response, they will be sent directly to the New Relic.

To see the sample of the PHP library, see the @arku31 implementation: https://github.com/arku31/roadrunner-newrelic

The special key which PHP may set to overwrite the transaction name is: `transaction_name`. For
example: `transaction_name:foo` means: set transaction name as `foo`. By default, `RequestURI` is used as the
transaction name.

```php
        $resp = new \Nyholm\Psr7\Response();
        $rrNewRelic = [
            'shopId:1', //custom data
            'auth:password', //custom data
            'transaction_name:test_transaction' //name - special key to override the name. By default it will use requestUri.
        ];

        $resp = $resp->withHeader('rr_newrelic', $rrNewRelic);
```
