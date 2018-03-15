# Kubernetes Logging with Fluent Bit (0.13-dev)

__DISCLAIMER__: The following 0.13-dev branch aims to be used for testing and development purposes only.

[Fluent Bit](http://fluentbit.io) is a lightweight and extensible __Log Processor__ that comes with full support for Kubernetes:

- Read Kubernetes/Docker log files from the file system or through Systemd Journal
- Enrich logs with Kubernetes metadata
- Deliver logs to third party storage services like Elasticsearch, InfluxDB, HTTP, etc.

This repository contains a set of Yaml files to deploy Fluent Bit which consider namespace, RBAC, Service Account, etc.

## Changelog

The following repository is having continuous updates, specifically in the Docker image that's being used.

- __0.13-dev:0.12, March 05th, 2018__
  - core: pack: add fast JSON validation
  - parser_decoder: extended support for 'escaped' mode and 'try_next' action

- __0.13-dev:0.11, March 05th, 2018__
  - core: pack: fix unnecessary packing on empty JSON tokens.

- __0.13-dev:0.10, March 01th, 2018__
  - filter_kubernetes: fix support for logs coming from Systemd.

- __0.13-dev:0.9, February 22th, 2018__
  - filter_kubernetes: improve handling of strings, objects, escaped strings, etc.
  - runtime tests: re-do tests for filter_kubernetes.

- __0.13-dev:0.8, February 14th, 2018__
  - core: new parser decoder feature (Decode_Field)
  - in_tail: check if a file is already being monitored
  - in_tail: new db.sync property
  - in_systemd: fix stalled reads
  - filter_record_modifier: support wildcard for Remove_key & Whitelist_key.

- __0.13-dev:0.7, February 06th, 2018__
  - core: fix memory spikes associated with tasks on pause/resume handlers.
  - in_tail: if a parser fails, pack raw text
  - filter_kubernetes: allow to use a custom pre-defined regex parser to gather metadata from file names (new __regex_parser__ option).
  - out_kafka: new configuration option rdkafka.x to configure rdkafka internals.

- __0.13-dev:0.6, January 29th, 2018__
  - http server: fix crash on Prometheus exporter

- __0.13-dev:0.5, January 29th, 2018__
  - lib: mbedtls: upgrade from v2.5.1 to 2.6.0.
  - filter_kubernetes
    - fix conditional when checking annotations.
  - in_tail
    - do not stop processing when NULL bytes are found.
    - double check file-rotation and fix possible memory leak introduced in previous changes.
  - out_forward
    - fix memory leak when talking to (old) Fluentd 0.12 servers.

- __0.13-dev:0.4, January 16th, 2018__
  - Initial public release

## What's new in 0.13-dev

We need your help testing the following new features available:

- Native Kafka output plugin
- HTTP endpoints (TCP port 2020):
  - /
  - /api/v1/metrics
  - /api/v1/metrics/prometheus
  - /api/v1/plugins
- Filter Kubernetes
  - Pod's can suggest a parser through annotations (e.g: logging.parser = apache)

## Getting started

[Fluent Bit](http://fluentbit.io) must be deployed as a DaemonSet, so on that way it will be available on every node of your Kubernetes cluster. To get started run the following commands to create the namespace, service account and role setup:

```
$ kubectl create namespace logging
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-service-account.yaml
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-role.yaml
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-role-binding.yaml
```

#### Fluent Bit to Elasticsearch

The next step is to create a ConfigMap that will be used by our Fluent Bit DaemonSet:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/0.13-dev/output/elasticsearch/fluent-bit-configmap.yaml
```

Fluent Bit DaemonSet ready to be used with Elasticsearch on a normal Kubernetes Cluster:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/0.13-dev/output/elasticsearch/fluent-bit-ds.yaml
```

#### Fluent Bit to Elasticsearch on Minikube

If you are using Minikube for testing purposes, use the following alternative DaemonSet manifest:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/0.13-dev/output/elasticsearch/fluent-bit-ds-minikube.yaml
```

#### Fluent Bit to Kafka

Kubectl reads files and URLs alike so you can apply the following based on a local clone:
```bash
BASE="."
```

or from a URL:
```bash
BASE="https://raw.githubusercontent.com/Yolean/fluent-bit-kubernetes-kafka/out-kafka"
```

Create namespace and RBAC resources according to Getting started above.
Then configure your Kafka bootstrap servers string in `$BASE/output/kafka/fluent-bit-configmap.yaml`.
The default is for [Yolean/kubernetes-kafka](https://github.com/Yolean/kubernetes-kafka).

```
kubectl apply -f $BASE/output/kafka/fluent-bit-configmap.yaml
```

Then depending on Kubernetes setup (the hostPath for logs differ slightly):

```
kubectl apply -f $BASE/output/kafka/fluent-bit-ds-minikube.yaml
# or
kubectl apply -f $BASE/output/kafka/fluent-bit-ds.yaml
```

## Details

The default configuration of Fluent Bit makes sure of the following:

- Consume all containers logs from the running Node.
- The [Tail input plugin](http://fluentbit.io/documentation/0.12/input/tail.html) will not append more than __5MB__  into the engine until they are flushed to the Elasticsearch backend. This limit aims to provide a workaround for [backpressure](http://fluentbit.io/documentation/0.12/configuration/backpressure.html) scenarios.
- The Kubernetes filter will enrigh the logs with Kubernetes metadata, specifically _labels_ and _annotations_. The filter only goes to the API Server when it cannot find the cached info, otherwise it uses the cache.
- The default backend in the configuration is Elasticsearch set by the [Elasticsearch Ouput Plugin](http://fluentbit.io/documentation/0.11/output/elasticsearch.html). It uses the Logstash format to ingest the logs. If you need a different Index and Type, please refer to the plugin option and do your own adjustments.
- There is an option called __Retry_Limit__ set to False, that means if Fluent Bit cannot flush the records to Elasticsearch it will re-try indefinitely until it succeed.

## Get back to us!

Your contribution on testing is highly appreciated, we aim to make logging cheaper for everybody so your feedback is fundamental, try to get back to us on:

- [Mailing List / Google Group](https://groups.google.com/forum/#!forum/fluent-bit)
- [Slack Channel #fluent-bit](http://slack.fluentd.org)
