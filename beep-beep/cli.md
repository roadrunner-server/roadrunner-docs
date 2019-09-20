# Server Commands
RoadRunner application can be started by calling a simple command from the root of your PHP application.

```
$ rr serve -v
```

You can also run RR in debug mode to view all incoming requests.

```
$ rr serve -v -d
```

You can force RR service to reload its HTTP workers.

```
$ rr http:reset
```

> You can attach this command as file watcher in your IDE.

To view the status of all active workers in interactive mode.

```
$ rr http:workers -i
```

```
+---------+-----------+---------+---------+--------------------+
|   PID   |  STATUS   |  EXECS  | MEMORY  |      CREATED       |
+---------+-----------+---------+---------+--------------------+
|    9440 | ready     |  42,320 | 31 MB   | 22 minutes ago     |
|    9447 | ready     |  42,329 | 31 MB   | 22 minutes ago     |
|    9454 | ready     |  42,306 | 31 MB   | 22 minutes ago     |
|    9461 | ready     |  42,316 | 31 MB   | 22 minutes ago     |
+---------+-----------+---------+---------+--------------------+
```
