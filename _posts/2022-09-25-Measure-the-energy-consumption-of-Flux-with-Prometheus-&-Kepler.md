---
layout: post
author: Niki Manoledaki
---

Use Flux to deploy Kepler and Prometheus to any Kubernetes cluster(s) and measure the energy consumption of any Kubernetes Pod or Node. 

To deploy Kepler and Prometheus, add their manifests to a Git repository. [Here](https://github.com/nikimanoledaki/gitops-energy-usage/tree/main/clusters) is a demo repository structure for how to do that. 

Flux will continuously reoncile and apply the Kepler and Prometheus manifests to your cluster. For example, by simply repeating the `flux bootstrap` command in a new cluster, Flux will sync with the repository and add Kepler and Prometheus to the new cluster.

To try this, create a local [kind](https://kind.sigs.k8s.io/) cluster (`kind create cluster`), bootstrap the cluster with Flux (`flux bootstrap github --owner=nikimanoledaki --repository=gitops-energy-usage --path=clusters`), wait for the resources to be created, then port-forward the `kube-prometheus` Pod. [This script](https://github.com/nikimanoledaki/gitops-energy-usage/blob/main/scripts/start-cluster.sh) performs all of these steps.

## Gather Pod-level energy metrics

Query the `kube-prometheus` HTTP API's `api/v1/query` endpoint with the following `curl` request:
```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=pod_curr_energy_in_pkg_millijoule{pod_namespace='flux-system'}" | jq
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "__name__": "pod_curr_energy_in_pkg_millijoule",
          "command": "helm-contr",
          "container": "kepler-exporter",
          "endpoint": "http",
          "instance": "kind-control-plane",
          "job": "kepler-exporter",
          "namespace": "kepler",
          "pod": "kepler-exporter-7nbzs",
          "pod_name": "helm-controller-9b6bb4f68-k2vp7",
          "pod_namespace": "flux-system",
          "service": "kepler-exporter"
        },
        "value": [
          1664036406.998,
          "1"
        ]
      },
      {
        "metric": {
          "__name__": "pod_curr_energy_in_pkg_millijoule",
          "command": "kustomize-",
          "container": "kepler-exporter",
          "endpoint": "http",
          "instance": "kind-control-plane",
          "job": "kepler-exporter",
          "namespace": "kepler",
          "pod": "kepler-exporter-7nbzs",
          "pod_name": "kustomize-controller-7f4687b878-65q97",
          "pod_namespace": "flux-system",
          "service": "kepler-exporter"
        },
        "value": [
          1664036406.998,
          "1"
        ]
      },
      {
        "metric": {
          "__name__": "pod_curr_energy_in_pkg_millijoule",
          "command": "source-con",
          "container": "kepler-exporter",
          "endpoint": "http",
          "instance": "kind-control-plane",
          "job": "kepler-exporter",
          "namespace": "kepler",
          "pod": "kepler-exporter-7nbzs",
          "pod_name": "source-controller-f8d655bdc-4fw6n",
          "pod_namespace": "flux-system",
          "service": "kepler-exporter"
        },
        "value": [
          1664036406.998,
          "1"
        ]
      },
      {
        "metric": {
          "__name__": "pod_curr_energy_in_pkg_millijoule",
          "command": "tini",
          "container": "kepler-exporter",
          "endpoint": "http",
          "instance": "kind-control-plane",
          "job": "kepler-exporter",
          "namespace": "kepler",
          "pod": "kepler-exporter-7nbzs",
          "pod_name": "notification-controller-7f5dbddc94-rtdhq",
          "pod_namespace": "flux-system",
          "service": "kepler-exporter"
        },
        "value": [
          1664036406.998,
          "1"
        ]
      }
    ]
  }
}
```
In the command above, the flag `--data-urlencode` is used to pass a Prometheus query in PromQL syntax. The result is piped to `jq` to transform it into json format. 

The query itself measures the energy consumption of all of the Pods in the `flux-system` namespace, where the Flux controllers are deployed.

There are four Flux controllers. These are the controllers for Sources, Helm, Kustomize, and Notification resources.

The command below returns the **sum** of the energy consumed by the Flux controllers:
```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=sum(pod_curr_energy_in_pkg_millijoule{pod_namespace='flux-system'})" | jq
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {},
        "value": [
          1664037856.556,
          "4"
        ]
      }
    ]
  }
}
```

Narrow down the json results with the following filters for `jq` to only return the value:
```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=sum(pod_curr_energy_in_pkg_millijoule{pod_namespace='flux-system'})" | jq '.data.result[0].value[0]'
1664037900.094
```

The value is `1664037900.094 mJ (millijoules)`. The section below, ["Joule for Beginners"](#joule-for-beginners), goes over the basics (or a refresher) of joules.

Ideally it would be great to narrow this further to a range that calculates the past ohur by using a [range vector](https://prometheus.io/docs/prometheus/latest/querying/basics/#range-vector-selectors).

## Gather Node-level energy metrics

The `node_enegy_stat` vector will return information on the Kubernetes Node:
```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=node_energy_stat" | jq
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "__name__": "node_energy_stat",
          "container": "kepler-exporter",
          "cpu_architecture": "Sandy Bridge",
          "endpoint": "http",
          "instance": "kind-control-plane",
          "job": "kepler-exporter",
          "namespace": "kepler",
          "node_block_devices_used": "20",
          "node_curr_bytes_read": "0",
          "node_curr_bytes_writes": "135168",
          "node_curr_cache_miss": "0",
          "node_curr_container_cpu_usage_seconds_total": "3",
          "node_curr_container_memory_working_set_bytes": "630784",
          "node_curr_cpu_cycles": "0",
          "node_curr_cpu_instr": "0",
          "node_curr_cpu_time": "0",
          "node_curr_energy_in_core_joule": "0.013000",
          "node_curr_energy_in_dram_joule": "0.095000",
          "node_curr_energy_in_gpu_joule": "0.000000",
          "node_curr_energy_in_other_joule": "0.000000",
          "node_curr_energy_in_pkg_joule": "0.013000",
          "node_curr_energy_in_uncore_joule": "0.000000",
          "node_name": "kind-control-plane",
          "pod": "kepler-exporter-7nbzs",
          "service": "kepler-exporter"
        },
        "value": [
          1664036361.77,
          "0"
        ]
      }
    ]
  }
}
```

Lastly, the json result for `node_energy_stat` narrowed down by adding filters to `jq`:
```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode "query=node_energy_stat" | jq '.data.result[0].value[0]'
1664037946.617
```

## Joule for beginners

What is a joule (J)? Very simply put, it is a unit of energy. 

1 joule equals 1000 millijoules (mJ), where `1 J = 1000 mJ`.

1 joule _per second_ equals to 1 Watt, so `1W = 1 J/s`.

Therefore, `1 W/h = 3600 J = 3600000 mJ`, which is the same as `(60 sec x 60 min) x 1000`, where the multiplication by 1000 converts joules to millijoules.



## Relevant docs / further reading:
- [Prometheus Docs: PromQL Expression Queries](https://prometheus.io/docs/prometheus/latest/querying/api/#expression-queries)
- [Example PromQL queries used to display Kepler on Grafana](https://github.com/sustainable-computing-io/kepler/blob/main/grafana-dashboards/Kepler-Exporter.json)
