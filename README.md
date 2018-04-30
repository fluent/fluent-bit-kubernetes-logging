# Kubernetes Logging with Fluent Bit v0.13



[Fluent Bit](http://fluentbit.io) is a lightweight and extensible __Log Processor__ that comes with full support for Kubernetes:

- Read Kubernetes/Docker log files from the file system or through Systemd Journal
- Enrich logs with Kubernetes metadata
- Deliver logs to third party storage services like Elasticsearch, InfluxDB, HTTP, etc.

This repository contains a set of Yaml files to deploy Fluent Bit which consider namespace, RBAC, Service Account, etc.

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
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-configmap.yaml
```

Fluent Bit DaemonSet ready to be used with Elasticsearch on a normal Kubernetes Cluster:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-ds.yaml
```

#### Fluent Bit to Elasticsearch on Minikube

If you are using Minikube for testing purposes, use the following alternative DaemonSet manifest:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-ds-minikube.yaml
```

#### Fluent Bit to Kafka

Create a ConfigMap that will be used by our Fluent Bit DaemonSet:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/kafka/fluent-bit-configmap.yaml
```

Fluent Bit DaemonSet ready to be used with Kafka on a normal Kubernetes Cluster:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/kafka/fluent-bit-ds.yaml
```

#### Fluent Bit to Elasticsearch on Minikube

If you are using Minikube for testing purposes, use the following alternative DaemonSet manifest:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-ds-minikube.yaml
```

## Details

The default configuration of Fluent Bit makes sure of the following:

- Consume all containers logs from the running Node.
- The [Tail input plugin](http://fluentbit.io/documentation/0.12/input/tail.html) will not append more than __5MB__  into the engine until they are flushed to the Elasticsearch backend. This limit aims to provide a workaround for [backpressure](http://fluentbit.io/documentation/0.13/configuration/backpressure.html) scenarios.
- The Kubernetes filter will enrigh the logs with Kubernetes metadata, specifically _labels_ and _annotations_. The filter only goes to the API Server when it cannot find the cached info, otherwise it uses the cache.
- The default backend in the configuration is Elasticsearch set by the [Elasticsearch Ouput Plugin](http://fluentbit.io/documentation/0.13/output/elasticsearch.html). It uses the Logstash format to ingest the logs. If you need a different Index and Type, please refer to the plugin option and do your own adjustments.
- There is an option called __Retry_Limit__ set to False, that means if Fluent Bit cannot flush the records to Elasticsearch it will re-try indefinitely until it succeed.

## Get back to us!

Your contribution on testing is highly appreciated, we aim to make logging cheaper for everybody so your feedback is fundamental, try to get back to us on:

- [Mailing List / Google Group](https://groups.google.com/forum/#!forum/fluent-bit)
- [Slack Channel #fluent-bit](http://slack.fluentd.org)
