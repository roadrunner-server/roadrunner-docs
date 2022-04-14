## PHP Client

[Roadrunner Worker](https://github.com/spiral/roadrunner-worker)

## Configuration

This plugin parse other's plugins configuration and uses flags to find the YAML config.

## Compatibility matrix

⚠️ Keep in mind that the `yaml` configuration version is not the same as the RR version. They have independent versions.

| RR version |                               Configuration version                           |
|-----------------------|--------------------------------------------------------------------|
|   **2.8+**             |          **2.7**                                                |
|   **2.7.x**   |          **2.7** `OR` Unversioned (treated as `v2.6.0`, will be auto-updated to `v2.7`) |
|   **<=2.6.x**   |          Doesn't support versions |

*non-versioned: configuration used in the 2.0.x-2.6.x releases.

## Flags

1. `-c [PATH]`: points RR to the configuration location.

## Minimal dependencies:

This plugin is independent.

## Worker sample:

```php
<?php

require __DIR__ . '/vendor/autoload.php';

// Create a new Worker from global environment
$worker = \Spiral\RoadRunner\Worker::create();

while ($data = $worker->waitPayload()) {
    // Received Payload
    var_dump($data);

    // Respond Answer
    $worker->respond(new \Spiral\RoadRunner\Payload('DONE'));
}
```

## Tips:

1. By default, `.rr.yaml` used as the configuration, located in the same directory with RR binary.

