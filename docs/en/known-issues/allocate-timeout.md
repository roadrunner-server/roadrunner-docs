# Allocate timeout error

RoadRunner allocates workers like usual processes (via `fork` + `exec` syscalls). While RoadRunner is in the process of creating a worker (connecting to the pipes, TCP, etc.), some part of the worker might freeze the initial handshake. RoadRunner waits `pool.allocate_timeout` time for handshake to complete and return this error if `pool.allocate_timeout` exceeded.

How to fix that?  

1. Check the `pool.allocate_timeout` option. It should be in the form of `pool.allocate_timeout: 1s` or `pool.allocate_timeout: 1h`, so you should specify the units of measurement.
2. xdebug might freeze the worker spawn while improperly configured. See this tutorial: https://roadrunner.dev/docs/php-debugging/2.x/en
