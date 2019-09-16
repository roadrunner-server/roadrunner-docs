# Production Usage
There are multiple tips and suggestions which must be acknowledged while running RoadRuner on production.

- database connections and any pipe/socket is the potential point of failure. Close all the connections after each iteration
- make sure to close all descriptors (especially in case of fatal exceptions)
- [optional] consider calling `gc_collect_cycles` after each execution if you want to keep the memory low (this will slow down your application a bit)
- watch memory leaks - you have to be more picky about what components you use. Workers will be restarted in case of a memory leak but it should not be hard to completely avoid this issue by properly designing your application
- watch the state pollution (i.e. globals or user data cache in memory)
- make sure NOT to listen 0.0.0.0 in RPC service (unless in Docker)
- connect to a worker using pipes for higher performance (Unix sockets just a bit slower)
- tweak your pool timings to the values you like
- a number of workers = number of CPU threads in your system, unless your application is IO bound, then pick the number heuristically 
- consider using `maxJobs` for your workers if you experience any issues with application stability over time
- RoadRunner is +40% performant using Keep-Alive connections
- setup memory limit at least 10-20% below max_memory_usage
- since RoadRunner workers run from cli you need to enable OPcache in CLI via `opcache.enable_cli=1`
