# Experimental Features

## Introduction
Starting from the RR `v2023.3.5` release, we have introduced a new feature called **Experimental Features**. This feature allows you to try out new features that are not yet ready for production use.

## How to enable experimental features
To enable experimental features, you need to run RR with the `-e` (`--experimental`) flag. For example:

```bash
./rr serve -e
```

Or:

```bash
./rr serve --experimental
```

## List of experimental features

### Support for the nested configurations. 

Using the following syntax, you may include other configuration files into the main one:

```yaml .rr.yaml
version: "3"

include:
  - .rr.include1-sub1.yaml
  - .rr.include1-sub2.yaml

reload:
  interval: 1s
  patterns: [".php"]
```
Where `.rr.include1-sub1.yaml` and `.rr.include1-sub2.yaml` are the configuration files that are located in the same directory as the main configuration file.
Includes override the main configuration file. For example, if you have the following nested configuration:

```yaml .rr.include1-sub1.yaml
version: "3"

server:
  command: "php php_test_files/psr-worker-bench.php"
  relay: pipes

http:
  address: 127.0.0.1:15389
  middleware:
    - "sendfile"
  pool:
    allocate_timeout: 10s
    num_workers: 2
```

It will override the `server` and `http` sections of the main configuration file. 
You may use env variables in the included configuration files, but you can't use overrides for the nested configuration. For example:

```yaml .rr.include1-sub1.yaml
version: "3"

server:
  command: "${PHP_COMMAND:-php_test_files/psr-worker-bench.php}"
  relay: pipes
```