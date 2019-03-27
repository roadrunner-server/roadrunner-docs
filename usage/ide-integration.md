# IDE Integration
RoadRunner uses PHP scripts in daemon mode, this means that you have to reload server every time you change your codebase. 

If you use any modern IDE you can achieve that by adding file watcher which automatically invokes command `rr http:reset`.

## In Docker
You can reset rr process in docker by connecting to it using local rr client. 

```yaml
rpc:
  listen: tcp://:6001
```

> Make sure to forward/expose port 6001.

Then run `rr http:reset` locally on file change.

> Another option is to use `maxJobs: 1` and to make rr destroy process after each request.
