# Installation

The easiest way to get the latest RoadRunner version is to use one of the pre-built release binaries which are available for
OSX, Linux, FreeBSD, and Windows. Instructions for using these binaries are on the GitHub [releases page](https://github.com/spiral/roadrunner/releases).

#### Building RoadRunner
RoadRunner can be compiled on Linux, OSX, Windows and other 64 bit environments as the only requirement is Go 1.11.

To get all needed dependencies:

```
$ go mod download
```

To build:

```
$ make
```

To test:

```
$ make test
```

## Quick Builds
You can also build your own version of RoadRunner locally (via Docker) using [the following tutorial](Quick-Builds).