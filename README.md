[![GoDoc](https://godoc.org/github.com/dm/logspout-logstash?status.svg)](https://godoc.org/github.com/dm/logspout-logstash)
[![Go Report Card](https://goreportcard.com/badge/dm/logspout-logstash)](https://goreportcard.com/report/dm/logspout-logstash)

I've restarted from the existing [looplab/logspout-logstash](https://github.com/looplab/logspout-logstash) and incorporated some existing forks' changes as well as my own. I've made this repo canonical to my own needs within Docker/Rancher for an ELK stack.

# logspout-logstash

A minimalistic adapter for [Logspout](https://github.com/gliderlabs/logspout) to write to Logstash. It also export some information about the Rancher environment it is running in if Rancher labels are available on the container as well as export all other Docker labels.

All labels not prefixed by `io.rancher` will be included with the `.` separator replaced by a `_` to prevent error in Elastic.

Started from the instructions [here](https://github.com/gliderlabs/logspout/blob/master/custom/README.md) on how to build your own Logspout container with custom modules.


## Docker Logspout-Logstash

Docker image for [dm/logspout-logstash](https://github.com/dm/logspout-logstash), available in docker hub [dmacedo/logspout-logstash:latest](https://hub.docker.com/r/dmacedo/logspout-logstash/)

### Docker Tags

`dmacedo/logspout-logstash` tagged images:

* `latest`: Latest stable release
* `v1.0.0`: First release v1.0.0

### Usage

Build docker image:
```bash
cd docker && make build
```

To run the image, set `LOGSTASH_HOST` env (defaults to `logstash.local`):
```bash
cd docker && LOGSTASH_HOST=logstash.local make run
```


## Additional documentation

Use by setting a docker environment variable `ROUTE_URIS=logstash://host:port` to the Logstash server.

The default protocol is UDP, but it is possible to change to TCP by adding `+tcp` after the logstash protocol when starting your container.

In your logstash config, set the input codec to `json` e.g:

```bash
input {
  udp {
    port  => 5000
    codec => json
  }
  tcp {
    port  => 5000
    codec => json
  }
}
```

## Available configuration options

For example, to get into the Logstash event's @tags field, use the `LOGSTASH_TAGS`
container environment variable. Multiple tags can be passed by using
comma-separated values.

```bash
  # Add any number of arbitrary tags to your event
  -e LOGSTASH_TAGS="docker,production"
```

The output into logstash should be like:

```json
    "tags": [
      "docker",
      "production"
    ],
```

You can also add arbitrary logstash fields to the event using the `LOGSTASH_FIELDS`
container environment variable:

```bash
  # Add any number of arbitrary fields to your event
  -e LOGSTASH_FIELDS="myfield=something,anotherfield=something_else"
```

The output into logstash should be like:

```json
    "myfield": "something",
    "another_field": "something_else",
```

Both configuration options can be set for every individual container, or for the logspout-logstash
container itself where they then become a default for all containers if not overridden there.

By setting the environment variable DOCKER_LABELS to a non-empty value, logspout-logstash will add all docker container labels as fields:

```json
"docker": {
    "hostname": "866e2ca94f5f",
    "id": "866e2ca94f5fe11d57add5a78232c53dfb6187f04f6e150ec15f0ae1e1737731",
    "image": "centos:7",
    "labels": {
        "a_label": "yes",
        "build-date": "20161214",
        "license": "GPLv2",
        "name": "CentOS Base Image",
        "pleasework": "okay",
        "some_label_with_dots": "more.dots",
        "vendor": "CentOS"
    },
    "name": "/ecstatic_murdock"
}
```

To be compatible with Elasticsearch, dots in labels will be replaced with underscores.

### Retrying

Two environment variables control the behavior of Logspout when the Logstash target isn't available:

`RETRY_STARTUP` causes Logspout to retry forever if Logstash isn't available at startup, and `RETRY_SEND` will retry sending log lines when Logstash becomes unavailable while Logspout is running.

Note that `RETRY_SEND` will work only if UDP is used for the log transport and the destination doesn't change; in any other case `RETRY_SEND` should be disabled, restart and reconnect instead and let `RETRY_STARTUP` deal with the situation.

With both retry options, log lines will be lost when Logstash isn't available. Set the environment variables to any nonempty value to enable retrying. The default is disabled.


This table shows all available configurations:

| Environment Variable | Input Type | Default Value |
|----------------------|------------|---------------|
| LOGSTASH_TAGS        | array      | None          |
| LOGSTASH_FIELDS      | map        | None          |
| DOCKER_LABELS        | any        | ""            |
| RETRY_STARTUP        | any        | ""            |
| RETRY_SEND           | any        | ""            |
