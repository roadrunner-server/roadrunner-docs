# Auto-Reloading
RoadRunner is able to automatically detect PHP file changes and reload connected services. Such approach allows you to develop, 
application without the `maxJobs: 1` or manual server reset.

## Configuration
To enable realoading for http service:

```yaml
# reload can reset rr servers when files change
reload:
  # refresh internval (default 1s)
  interval: 1s

  # file extensions to watch, defaults to [.php]
  patterns: [".php"]

  # list of services to watch
  services:
    http:
      # list of dirs, "" root
      dirs: [""]

      # include sub directories
      recursive: true
```

## Performance
The `reload` component will affect the performance of application server. Make sure to use it in development mode only.
