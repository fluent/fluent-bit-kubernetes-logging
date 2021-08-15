# Kubernetes Logging with Fluent Bit

[Fluent Bit](http://fluentbit.io) is a lightweight and extensible __Log Processor__ that comes with full support for Kubernetes:

- Read Kubernetes/Docker log files from the file system or through systemd Journal
- Enrich logs with Kubernetes metadata
- Deliver logs to third party storage services like Elasticsearch, InfluxDB, HTTP, etc.

This repository contains a set of Yaml files to deploy Fluent Bit which consider namespace, RBAC, Service Account, etc.

## Getting started

[Fluent Bit](http://fluentbit.io) must be deployed as a DaemonSet so that it will be available on every node of your Kubernetes cluster. To get started run the following commands to create the namespace, service account and role setup:

```
$ kubectl create namespace logging
$ kubectl apply -k ./base
```

(`./base` can be replaced with a URL if [#90](https://github.com/fluent/fluent-bit-kubernetes-logging/pull/90) gets merged)

If you are deploying fluent-bit on openshift, you additionally need to run:

```
$ kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-openshift-security-context-constraints.yaml
```

The base only configures [Stdout](https://docs.fluentbit.io/manual/pipeline/outputs/standard-output) output.
Log records are found using `kubectl logs` on fluentbit pods.
This is a good way to experiment with configuration.
Switch to [Format](https://docs.fluentbit.io/manual/pipeline/outputs/standard-output#configuration-parameters) `msgpack` to also see the "tag" (helpful when developing your pipeline).

## Logs ingested to Loki

This repository serves as example of Kubernetes yaml for a log processing stack,
not as example of [how](https://docs.fluentbit.io/manual/concepts/data-pipeline) Fluent-bit can process, refine and forward log entries.

Use the [loki](./loki) base to replace Stdout with forwarding to a standalone Loki instance. Loki can do some processing at [query time](https://grafana.com/docs/loki/latest/logql/) which helps keep this repo light on pipeline config.

The [loki-plugin](./loki-plugin) base uses an out-of-tree plugin with `DropSingleKey`
so that the log records forwarded are the actual logged lines from pods,
but still queryable on labels like `pod` and `container`.

See the [lokitest](./loki/test/lokitest-job.yaml) Job for some examples of how to consume logs through Loki.

The loki folders also serve as [example](./loki/kustomization.yaml) of how to use Kustomize to override selected parts of the `base`'s *.conf.

## Get in touch with us!

Your contribution to testing is highly appreciated. We aim to make logging cheaper for everybody so your feedback is fundamental. Please get in touch on:

- [Mailing List / Google Group](https://groups.google.com/forum/#!forum/fluent-bit)
- [Slack Channel #fluent-bit](http://slack.fluentd.org)
