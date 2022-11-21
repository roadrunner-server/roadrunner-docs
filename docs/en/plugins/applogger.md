# Application logger

This plugin might be used when the user needs to send a logs from the worker with a different severity levels. Moreover,
the user might use this plugin to send a raw messages to the RR `STDERR`.

## API

####
- [PHP Library](https://github.com/roadrunner-php/app-logger)

#### RPC Interface 

All methods accept a `string` (which will be logger) as a first argument, and `bool` placeholder for the second arg.

```go
func (r *RPC) Error(in string, _ *bool) error {}

func (r *RPC) Info(in string, _ *bool) error {}

func (r *RPC) Warning(in string, _ *bool) error {}

func (r *RPC) Debug(in string, _ *bool) error {}

func (r *RPC) Log(in string, _ *bool) error {}
```

Methods represent severity levels. Where the `Log` method doesn't have a severity and logger directly to the `STDERR`. 
