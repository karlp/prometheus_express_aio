# prometheus_express

A [Prometheus](https://prometheus.io/) SDK for CircuitPython/MicroPython boards, allowing sensor data to be integrated
into existing Prometheus/Grafana monitoring infrastructure.

- only depends on `socket`
- API-compatible with [prometheus/client_python](https://github.com/prometheus/client_python)
- basic HTTP server
- not terribly slow (`wrk` reports upwards of 100rps with 2 metrics)

## Contents

- [prometheus_express](#prometheusexpress)
  - [Contents](#contents)
  - [Supported Hardware](#supported-hardware)
  - [Supported Features](#supported-features)
    - [HTTP](#http)
    - [Labels](#labels)
    - [Metric Types](#metric-types)
      - [Counter](#counter)
      - [Gauge](#gauge)
    - [Registries](#registries)
  - [Planned Features](#planned-features)
  - [Known Issues](#known-issues)
    - [Load Causes OSError](#load-causes-oserror)

## Supported Hardware

This library is developer for the [Adafruit Feather M4 Express](https://www.adafruit.com/product/3857) running
MicroPython 4.1.0 or better, with an [Adafruit Ethernet FeatherWing](https://www.adafruit.com/product/3201) attached.

## Supported Features

### HTTP

This module implements a very rudimentary HTTP server that likely violates some part of the spec. However, it works
with Chrome, curl, and Prometheus itself.

Call `start_http_server(port)` to bind a socket and begin listening.

Call `await_http_request(server, registry)` to await an incoming HTTP request and respond with the metrics in
`registry`.

### Labels

Labels are not yet implemented.

### Metric Types

Currently, `Counter` and `Gauge` are the only metric types implemented.

#### Counter

Both `inc` and `dec` are implemented.

#### Gauge

Extends [counter](#counter) with `set`.

### Registries

Registries may be created with a namespace: `CollectorRegistry(namespace='foo')`

Call `registry.print()` to format metrics in Prometheus'
[plain text exposition format](https://prometheus.io/docs/instrumenting/exposition_formats/#text-based-format):

```
# HELP prom_express_test_counter a test counter
# TYPE prom_express_test_counter counter
prom_express_test_counter 1588
# HELP prom_express_test_gauge a test gauge
# TYPE prom_express_test_gauge gauge
prom_express_test_gauge 3887
```

Metrics may be registered with multiple registries.

## Planned Features

- respect request path, only respond to `/metrics`
- additional metric types (Histogram, Summary)
- push metrics

## Known Issues

### Load Causes OSError

Load testing the HTTP endpoint may cause one of a variety of `OSError`s, often `errno` 3, 4, or 7.

Not sure what is causing the errors, but it is not predictable and may not appear immediately:

```shell
> ./wrk -c 1 -d 60s -t 1 http://server:8080/
Running 1m test @ http://server:8080/
  1 threads and 1 connections
^C  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     8.64ms  485.57us  12.81ms   97.75%
    Req/Sec   111.60      5.21   121.00     71.00%
  2222 requests in 20.61s, 671.83KB read
  Socket errors: connect 0, read 1, write 2743, timeout 0
Requests/sec:    107.82
Transfer/sec:     32.60KB
```

Some are fatal:

```
Connection from ('client', 8080)
Accepting...
Connection from ('client', 8080)
Accepting...
Error accepting request: [Errno 128] ENOTCONN
Binding: server:8080
Traceback (most recent call last):
  File "code.py", line 90, in <module>
  File "code.py", line 87, in main
  File "code.py", line 87, in main
  File "code.py", line 55, in bind
  File "/lib/prometheus_express/http.py", line 11, in start_http_server
OSError: 4
```

Others require the socket to be rebound:

```
Connection from ('client', 8080)
Accepting...
Error accepting request: 7
Binding: server:8080
Accepting...
```
