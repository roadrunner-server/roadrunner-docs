# Environment configuration
All RoadRunner workers will inherit the system configuration available for the parent server process. In addition, you can 
customize the set of env variables to be passed to your workers using part `env` of `.rr` configuration file.

```yaml
server:
  command: "php worker.php"
  env:
     key: value
```

> All keys will be automatically uppercased!

### Using environment variables in the configuration

- Option 1:

```bash
set -a
source /var/www/config/.env
set +a

exec /var/www/rr \
  -c /var/www/.rr.yaml \
  -w /var/www \  
  -o http.pool.num_workers=${RR_NUM_WORKERS:-8} \
  -o http.pool.max_jobs=${RR_MAX_JOBS:-16} \
  -o http.pool.supervisor.max_worker_memory=${RR_MAX_WORKER_MEMORY:-512}
  serve
  ```
  Where: 
  - `-w`: is working directory.
  - `-o`: is option to overwrite. All options might be overwritten.
  - `/var/www/config/.env`: contains needed env variables.
  - `${RR_NUM_WORKERS:-8}`: Use `RR_NUM_WORKERS` or 8 by default (if no there are no `RR_NUM_WORKERS` in the `.env`)

---

  - Option 2:

```yaml
http:
  address: 127.0.0.1:15389
  middleware: [ gzip ] 
  pool:
    num_workers: ${RR_NUM_WORKERS}
    max_jobs: ${RR_MAX_JOBS}
```

RR is able to expand the environment variable from the sentence like `${xxx}` OR `$xxx`

### Default ENV values inside the worker
RoadRunner provides set of ENV values to help the PHP process to identify how to properly communicate with the server.

Key      | Description
---      | ---
RR_MODE  | Identifies what mode worker should work with (`http`, `temporal`, `grpc`, `jobs`, `tcp`)
RR_RPC   | Contains RPC connection address when enabled.
RR_RELAY | "pipes" or "tcp://...", depends on server relay configuration.
