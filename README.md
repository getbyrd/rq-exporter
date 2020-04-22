# Python RQ Prometheus Exporter

Prometheus metrics exporter for the RQ (Redis Queue) job queue library.

## Exported Metrics

**RQ Metrics:**

* `rq_workers`: RQ workers

    * **Type**: Gauge
    * **Labels**: `name`, `queues`, `state`

    Example:

    ```
    rq_workers{name="40d33ed9541644d79373765e661b7f38", queues="default", state="idle"} 1.0
    rq_workers{name="fe9a433575e04685a53e4794b2eaeea9", queues="high,default,low", state="busy"} 1.0
    ```

* `rq_jobs`: RQ jobs by queue and status

    * **Type**: Gauge
    * **Labels**: `queue`, `status`

    Example:

    ```
    rq_jobs{queue="default", status="queued"} 2.0
    rq_jobs{queue="default", status="started"} 1.0
    rq_jobs{queue="default", status="finished"} 5.0
    rq_jobs{queue="default", status="failed"} 1.0
    rq_jobs{queue="default", status="deferred"} 1.0
    rq_jobs{queue="default", status="scheduled"} 2.0
    ```

**Request processing metrics:**

* `rq_request_processing_seconds_count`: Number of requests processed (Scrape counts)

    * **Type**: Summary

* `rq_request_processing_seconds_sum`: Total sum of time in seconds processing the requests

    * **Type**: Summary

* `rq_request_processing_seconds_created`: Time created at (`time.time()` return value)

    * **Type**: Gauge

## Configuration

Environment variables:

`RQ_REDIS_URL`: Redis URL in the form `redis://:[password]@[host]:[port]/[db]`

Or you can use these variables instead:

* `RQ_REDIS_HOST`: Redis host name (default: `localhost`)
* `RQ_REDIS_PORT`: Redis port number (default: `6379`)
* `RQ_REDIS_DB`: Redis database number (default: `0`)
* `RQ_REDIS_AUTH`: Redis password (default: `None`)
* `RQ_REDIS_AUTH_FILE`: Redis password file (e.g. Path of a mounted Docker secret)

When `RQ_REDIS_AUTH_FILE` is set `RQ_REDIS_AUTH` will be ignored.

## Starting a WSGI Server

```console
# Start a WSGI server on port 8000
$ python -m rq_exporter
```

To start the server on a different port

```console
$ python -m rq_exporter 8080
```

## Using With Gunicorn

The WSGI application instance is available at `rq_exporter.app`:

```console
$ gunicorn rq_exporter:app -b 0.0.0.0:8000
```

**Note about concurrency**:

The exporter is going to work without any problems with multiple workers but you will get different values for these metrics:

* `rq_request_processing_seconds_count`
* `rq_request_processing_seconds_sum`
* `rq_request_processing_seconds_created`

This is fine if you don't care about these metrics, these are only for measuring the count and time processing the RQ data, so the other RQ metrics are not going to be affected.

But you can still use multiple threads with 1 worker process to handle multiple concurrent requests:

```console
$ gunicorn rq_exporter:app -b 0.0.0.0:8000 --threads 2
```

## Building the Docker Image

```console
$ # Build the docker image and tag it rq_exporter:latest
$ docker build -t rq_exporter .
$ # Run the image after the build has completed
$ docker run -it rq_exporter
$ # Override Gunicorn command line options
$ docker run -it rq_exporter -b 0.0.0.0:8080 --log-level debug --threads 2
$ # Provide environment variables
$ docker run -it -e RQ_REDIS_HOST=redis -e RQ_REDIS_AUTH=123456 rq_exporter
```

The image can also be built using `docker-compose`:

```console
$ docker-compose build
```

Check out the `docker-compose.yml` file for usage example.
