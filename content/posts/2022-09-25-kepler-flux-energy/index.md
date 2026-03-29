---
title: "Measure the Energy Consumption of Flux with Prometheus and Kepler"
date: 2022-09-25
tags: ["kubernetes", "kepler", "prometheus", "sustainability", "flux"]
---

[Kepler](https://github.com/sustainable-computing-io/kepler) can be used with [Prometheus](https://prometheus.io/) to measure the energy consumption of any Kubernetes Pod, Node, or Namespace. Use [Flux](https://fluxcd.io/) to deploy Kepler the [GitOps](https://www.weave.works/technologies/gitops/) way, and then use Kepler to measure the energy consumption of Flux!

## Deploy Kepler with Flux

To deploy Kepler and Prometheus, add their manifests to a Git repository. [Here](https://github.com/nikimanoledaki/gitops-energy-usage/tree/main/clusters) is a demo repository structure for how to do that.

Flux will continuously reconcile and apply the Kepler and Prometheus manifests to your cluster. For example, by simply repeating the `flux bootstrap` command in a new cluster, Flux will sync with the repository and add Kepler and Prometheus to the new cluster.

To try this, create a local [kind](https://kind.sigs.k8s.io/) cluster (`kind create cluster`), bootstrap the cluster with Flux (`flux bootstrap github --owner=nikimanoledaki --repository=gitops-energy-usage --path=clusters`), wait for the resources to be created, then port-forward the `kube-prometheus` Pod. [This script](https://github.com/nikimanoledaki/gitops-energy-usage/blob/main/scripts/start-cluster.sh) performs all of these steps.

## Gather Pod-level energy metrics

Query the `kube-prometheus` HTTP API's `api/v1/query` endpoint with the following `curl` request:

```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=pod_curr_energy_in_pkg_millijoule{pod_namespace='flux-system'}" | jq
```

The flag `--data-urlencode` passes a Prometheus query in PromQL syntax. The result is piped to `jq` to transform it into JSON format.

The query measures the energy consumption of all of the Pods in the `flux-system` namespace, where the Flux controllers are deployed.

The command below returns the **sum** of the energy consumed by the Flux controllers:

```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=sum(pod_curr_energy_in_pkg_millijoule{pod_namespace='flux-system'})" | jq
```

Narrow down the JSON results with `jq` filters to return only the value:

```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=sum(pod_curr_energy_in_pkg_millijoule{pod_namespace='flux-system'})" | jq '.data.result[0].value[0]'
```

## Gather Node-level energy metrics

The `node_energy_stat` vector returns information on the Kubernetes Node:

```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=node_energy_stat" | jq
```

## Joule for beginners

A joule (J) is a unit of energy.

1 joule equals 1000 millijoules (mJ): `1 J = 1000 mJ`.

1 joule per second equals 1 Watt: `1W = 1 J/s`.

Therefore: `1 W/h = 3600 J = 3600000 mJ`, which is `(60 sec x 60 min) x 1000`.

## Further reading

- [Prometheus Docs: PromQL Expression Queries](https://prometheus.io/docs/prometheus/latest/querying/api/#expression-queries)
- [Example PromQL queries used to display Kepler on Grafana](https://github.com/sustainable-computing-io/kepler/blob/main/grafana-dashboards/Kepler-Exporter.json)
