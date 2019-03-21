# AWS Lambda
RoadRunner can run PHP as AWS Lambda function.

### Installation
Prior to the function deployment, you must compile or download PHP binary files to run your application. There are multiple projects available for such goal:

- https://github.com/araines/serverless-php
- https://github.com/stechstudio/php-lambda (includes pre-built versions of PHP binaries)

Place PHP binaries in a `bin/` folder of your project.

### PHP Worker
PHP worker does not require any specific configuration to run inside Lambda function. We can use default snippet with internal counter to demonstrate how workers are being reused:

```php
<?php
/**
 * @var Goridge\RelayInterface $relay
 */
use Spiral\Goridge;
use Spiral\RoadRunner;

ini_set('display_errors', 'stderr');
require 'vendor/autoload.php';

$rr = new RoadRunner\Worker(new Goridge\StreamRelay(STDIN, STDOUT));

$count = 0;
while ($body = $rr->receive($context)) {
    try {
        $count ++;
        $rr->send((string)$body . ":" . $count, (string)$context);
    } catch (\Throwable $e) {
        $rr->error((string)$e);
    }
}
```

Name this file `handler.php` and put it into root of your project. Make sure to run `composer require spiral/roadrunner`.

### Application
We can create a simple application to demonstrate how it works:

```golang
package main

import (
	"context"
	"github.com/aws/aws-lambda-go/lambda"
	"github.com/spiral/roadrunner"
	"os"
	"time"
)

var srv *roadrunner.Server

func init() {
	os.Setenv("PATH", os.Getenv("PATH")+":"+os.Getenv("LAMBDA_TASK_ROOT"))
	os.Setenv("LD_LIBRARY_PATH", "./lib:/lib64:/usr/lib64")

	srv = roadrunner.NewServer(
		&roadrunner.ServerConfig{
			Command: "bin/php handler.php",
			Relay:   "pipes",
			Pool: &roadrunner.Config{
				NumWorkers:      1,
				MaxJobs:         100,
				AllocateTimeout: time.Second,
				DestroyTimeout:  time.Second,
			},
		},
            )
}

func main() {
	if err := srv.Start(); err != nil {
		panic(err)
	}
	defer srv.Stop()

	lambda.Start(handle)
}

func handle(ctx context.Context, input string) (string, error) {
	res, err := srv.Exec(&roadrunner.Payload{Body: []byte(input)})

	return res.String(), err
}
```

To build and package your lambda function run:

```
$ GOOS=linux GOARCH=amd64 go build -o main main.go 
$ zip main.zip * -r
```

You can now upload and invoke your handler using simple string event.

## Notes
There are multiple notes you have to acknowledge.

- start with 1 worker per lambda function in order to control your memory usage.
- make sure to include env variables listed in the code to properly resolve the location of PHP binary and it's dependencies.
- avoid database connections without concurrency limit
- avoid database connections