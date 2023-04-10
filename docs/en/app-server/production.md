# Production Usage

While running RoadRunner in production, there are several tips and suggestions that need to be considered.

## State and memory

State and memory are **not shared** between different worker instances, but are **shared** for a single worker instance.
Since a single worker typically processes more than a single request, you should be careful about this:

- Make sure you close all descriptors (especially on fatal exceptions).
- [optional] Consider calling `gc_collect_cycles` after every execution if you want to keep memory usage low
  (this will slow down your application a bit).
- Watch out for memory leaks - you need to be more selective about the components you use. Workers will be restarted in case of a
  memory leak, but it should not be difficult to avoid this problem altogether by designing your application properly.
- Avoid state pollution (i.e., caching globals or user data in memory).
- Database connections and any pipe/socket is the potential point of failure. An easy way to deal with this is to close
  all connections after every iteration. Note that this is not the most performant solution.
  
## Useful Tips

- Make sure you are NOT listening on 0.0.0.0 in the RPC service (unless in Docker).
- Connect to a worker using pipes for better performance (Unix sockets are just a bit slower).
- Adjust your pool timings to the values you like.
- Number of workers = number of CPU threads in your system, unless your application is IO bound, then choose the number heuristically.
- Consider using `max_jobs` for your workers if you experience application stability memory issues over time.
- RoadRunner has +40% performance when using keep-alive connections.
- Set the memory limit at least 10-20% below `max_memory_usage`.
- Since RoadRunner runs workers from cli, you need to enable OPcache in the CLI with `opcache.enable_cli=1`.
- Make sure to use [health check endpoint](../lab/health.md) when running rr in a cloud environment.
- Use the `user` option in the `server` plugin configuration to start worker processes from the specified user on Linux-based systems. Note that in this case RoadRunner should be started from the `root` to allow fork-exec processes from different users.
- If your application uses mostly IO (disk, network, etc), you can allocate as many workers as you have memory for the application. Workers are cheap. A hello-world worker uses no more than ~26Mb of RSS memory.
- For CPU bound operation, see an average CPU load and choose the number of workers to consume 90-95% CPU. Leave a few percent for the GC of the GO (not necessary btw).
- If you have ~const workers latency, you can calculate the number of workers needed to handle the target [load] (https://github.com/spiral/roadrunner/discussions/799#discussioncomment-1332646).
