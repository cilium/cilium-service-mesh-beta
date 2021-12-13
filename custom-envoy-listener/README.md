# Example Custom Envoy Listener: Prometheus Metrics

This example replicates the Prometheus metrics listener which is
already available via Cilium Agent `--proxy-prometheus-port` command
line option. So the point of this example is not to add new
functionality, but to show how a feature that previously required
Cilium Agent code changes can be implemented with the new Cilium Envoy
Config CRD.


## Install Cilium and Hubble with Service Mesh Beta Support

[Install Cilium to your test cluster](../INSTALLATION.md) and run
`cilium version` to check that you have version 0.10.0 or above.


## Apply Example CRD

This example adds a new Envoy listener
`envoy-prometheus-metrics-listener` on the standards Prometheus port
(`9090`) to each Cilium node, translating the default Prometheus
metrics path `/metrics` to Envoy's Prometheus metrics path
`/stats/prometheus`.

Apply this Cilium Envoy Config CRD:

```
kubectl apply -f https://github.com/cilium/cilium-service-mesh-beta/blob/main/custom-envoy-listener/envoy-prometheus-metrics-listener.yaml
```

Let's take a look at the Cilium Envoy Config CRD. It begins with this
preamble:

```
apiVersion: cilium.io/v2alpha1
kind: CiliumEnvoyConfig
metadata:
  name: envoy-prometheus-metrics-listener
```

This specifies the API version as `cilium.io/v2alpha1`, meaning that
this API is subject to change before it will be added to
`cilium.io/v2`.

`kind` spells out the new Cilium Envoy Config type (`CiliumEnvoyConfig`).

The only metadata needed is the name for the resource (here
`envoy-prometheus-metrics-listener`. This version of the CEC CRD is
Cluster-scoped, (i.e., not namespaced), so the name needs to be unique
in the cluster, unless you want to replace a CRD with a new one.

This CEC CRD `spec` only contains two Envoy resources:
```
spec:
  resources:
  - "@type": type.googleapis.com/envoy.config.listener.v3.Listener
    name: envoy-prometheus-metrics-listener
    address:
      socket_address:
        address: "127.0.0.1"
        ipv4_compat: true
        port_value: 9090
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: envoy-prometheus-metrics-listener
          rds:
            route_config_name: prometheus_route
          http_filters:
          - name: envoy.filters.http.router
  - "@type": type.googleapis.com/envoy.config.route.v3.RouteConfiguration
    name: prometheus_route
    virtual_hosts:
    - name: "prometheus_metrics_route"
      domains: ["*"]
      routes:
      - match:
          path: "/metrics"
        route:
          cluster: "envoy-admin"
          prefix_rewrite: "/stats/prometheus"
```

Note that these Envoy resources are not validated by k8s at all, so
any errors in the Envoy resources will only be seen by the Cilium
Agent observing these CRDs. This means that `kubectl apply` will
report success, while parsing and/or installing the resources to the
node-local Envoy instance may have failed. Currently the only way of
verifying this is by observing Cilium Agent logs for errors and
warnings:

```
kubectl logs -n kube-system cilium-xxxxx | grep -E "level=(error|warning)"
```

Please replace `cilium-xxxxx` with one of the Cilium Agent pod names
you see in `kubectl get pods -A`.

This example contains two Envoy resources, one `Listener` and one
`RouteConfiguration`. Each of the resources needs to have a unique
name. Envoy resource names for different resource types can be the
same, but they are still separated by the resource type.

Note that same Envoy resource names for a given resource type in
different CEC CRDs will overwrite each other. So for now you need make
sure the resource names are different in different CEC CRDs you may
deploy. This limitation may be lifted in future versions e.g., by
namespacing the resource names with the CEC CRD name.


## Supported Envoy API Versions

As of now only the Envoy API v3 is supported.


## Supported Envoy Extension Resource Types

Envoy extensions are resource types that may or may not be built in to
an Envoy build. The standard types referred to in Envoy documetation,
such as `type.googleapis.com/envoy.config.listener.v3.Listener`, and
`type.googleapis.com/envoy.config.route.v3.RouteConfiguration`, are
always available.

Cilium nodes deploy an Envoy image to support Cilium HTTP policy
enforcement and observability. This build of Envoy has been optimized
for the needs of the Cilium Agent and does not contain many of the
Envoy extensions available in the Envoy code base.

To see which Envoy extensions are available, please have a look at
[this Envoy extensions configuration
file](https://github.com/cilium/proxy/blob/master/envoy_build_config/extensions_build_config.bzl).
Only the extensions that have not been commented out with `#` are
built in to the Cilium Envoy image. Currently this contains the
following extensions:

- `envoy.filters.http.router`
- `envoy.filters.listener.tls_inspector`
- `envoy.filters.network.http_connection_manager`
- `envoy.filters.network.mongo_proxy`
- `envoy.filters.network.mysql_proxy`
- `envoy.filters.network.tcp_proxy`
- `envoy.stat_sinks.metrics_service`
- `envoy.transport_sockets.raw_buffer`

We will evolve the list of built in extensions based on customer
feedback for those extensions to be available to be used from CEC
CRDs.

## Test the Listener Port

Test that the new port is responding to the metrics requests:

```
curl http://<node-IP>:9090/metrics
```

Where `<node-IP>` is the IP address of one of your k8s cluster nodes.

## Clean-up

Remove the prometheus listener with:

```
kubectl delete -f https://github.com/cilium/cilium-service-mesh-beta/blob/main/custom-envoy-listener/envoy-prometheus-metrics-listener.yaml
```
