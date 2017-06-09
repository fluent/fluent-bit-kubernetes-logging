# Help to Test!

[Fluent Bit](http://fluentbit.io) v0.11 was just released at the end of March 2017 and it comes with support for Kubernetes logging needs: it can consume logs and enrich them with proper Kubernetes metadata, retrieved from the API server of course.

Our goal is to make logging cheaper in terms of memory consumption. In order to accomplish this we need to deal with different factors, not only processing incoming logs but also _formatting_ the data for our backends like Elasticsearch, which can be a little expensive as it requires a JSON representation.

> Note: Fluent Bit uses a binary representation for logs, converting this to JSON requires enough memory for the process.

## Getting started

Fluent Bit must be deployed as a DaemonSet, on that way it will be available on every node of your Kubernetes cluster.

This repository contains two Yaml DaemonSet files:

| Yaml file | Description |
|-----------|-------------|
| [fluent-bit-daemonset-elasticsearch](fluent-bit-daemonset-elasticsearch.yaml) | deploys a stable release of Fluent Bit. |

The current DaemonSet points to this specific Docker Hub image:

[0.11](https://hub.docker.com/r/fluent/fluent-bit-kubernetes-daemonset/tags/) fluent/fluent-bit-kubernetes-daemonset:0.11

### Steps

1. Make sure your Elasticsearch backend is running and can be reach through the hostname _elasticsearch-logging_. This value can be changed in the Yaml file

```
wget https://github.com/kubernetes/kubernetes/raw/master/cluster/addons/fluentd-elasticsearch/es-controller.yaml
wget https://github.com/kubernetes/kubernetes/raw/master/cluster/addons/fluentd-elasticsearch/es-service.yaml
kubectl create -f es-controller.yaml
kubectl create -f es-service.yaml
```

2. Deploy the daemonset file from this repository:

```bash
$ kubectl apply -f fluent-bit-daemonset-elasticsearch.yaml
```

## Details

The default configuration of Fluent Bit makes sure of the following:

- Consume all containers logs from the running Node.
- The [Tail input plugin](http://fluentbit.io/documentation/0.11/input/tail.html) will not append more than __5MB__  into the engine until they are flushed to the Elasticsearch backend. This limit aims to provide a workaround for [backpressure](http://fluentbit.io/documentation/0.11/configuration/backpressure.html) scenarios.
- The Kubernetes filter will enrigh the logs with Kubernetes metadata, specifically _labels_ and _annotations_. The filter only goes to the API Server when it cannot find the cached info, otherwise it uses the cache.
- The default backend in the configuration is Elasticsearch set by the [Elasticsearch Ouput Plugin](http://fluentbit.io/documentation/0.11/output/elasticsearch.html). It uses the Logstash format to ingest the logs. If you need a different Index and Type, please refer to the plugin option and do your own adjustments.
- There is an option called __Retry_Limit__ set to False, that means if Fluent Bit cannot flush the records to Elasticsearch it will re-try indefinitely until it succeed.

## Get back to us!

Your contribution on testing is highly appreciated, we aim to make logging cheaper for everybody so your feedback is fundamental, try to get back to us on:

- [Mailing List / Google Group](https://groups.google.com/forum/#!forum/fluent-bit)
- [Slack Channel #fluent-bit](http://slack.fluentd.org)
