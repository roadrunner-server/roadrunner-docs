# About RoadRunner
[![Latest Stable Version](https://poser.pugx.org/spiral/roadrunner/version)](https://packagist.org/packages/spiral/roadrunner)
[![GoDoc](https://godoc.org/github.com/spiral/roadrunner?status.svg)](https://godoc.org/github.com/spiral/roadrunner)
[![Build Status](https://travis-ci.org/spiral/roadrunner.svg?branch=master)](https://travis-ci.org/spiral/roadrunner)
[![Go Report Card](https://goreportcard.com/badge/github.com/spiral/roadrunner)](https://goreportcard.com/report/github.com/spiral/roadrunner)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/spiral/roadrunner/badges/quality-score.png)](https://scrutinizer-ci.com/g/spiral/roadrunner/?branch=master)
[![Codecov](https://codecov.io/gh/spiral/roadrunner/branch/master/graph/badge.svg)](https://codecov.io/gh/spiral/roadrunner/)

RoadRunner is an open-source (MIT licensed), high-performance PHP application server, load balancer, and process manager. It supports running as a service with the ability to extend its functionality on a per-project basis. RoadRunner includes PSR-7 compatible HTTP server.

Features:
--------
- production-ready
- PSR-7 HTTP server (file uploads, error handling, static files, hot reload, middlewares, event listeners)
- HTTPS and HTTP/2 support (including HTTP/2 Push, H2C)
- fully customizable server
- flexible environment configuration
- no external PHP dependencies (64bit version required), drop-in (based on [Goridge](https://github.com/spiral/goridge))
- load balancer, process manager and task pipeline
- frontend agnostic ([Queue](https://github.com/spiral/jobs), PSR-7, [GRPC](https://github.com/spiral/php-grpc), etc)
- integrated metrics (Prometheus)
- works over TCP, UNIX sockets and standard pipes
- automatic worker replacement and safe PHP process destruction
- worker create/allocate/destroy timeouts
- max jobs per worker
- worker lifecycle management (controller) 
    - maxMemory (graceful stop)
    - TTL (graceful stop)
    - idleTTL (graceful stop)
    - execTTL (brute, max_execution_time)   
- payload context and body
- protocol, worker and job level error management (including PHP errors)
- very fast (~250k rpc calls per second on Ryzen 1700X using 16 threads)
- integrations with Symfony, Laravel, Slim, CakePHP, Zend Expressive, Spiral
- works on Windows

License:
--------
The MIT License (MIT). Please see [`LICENSE`](license.md) for more information. By SpiralScout.
