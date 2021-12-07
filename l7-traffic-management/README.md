- [ ] Provide images: cilium-cli, cilium, operator-generic, hubble-relay 
- [ ] Update instructions to use the supplied beta images 
- [ ] Draw a diagram 

# Layer 7 Traffic Management

Cilium Service Mesh defines a `CiliumEnvoyConfig` CRD which allows users to set the configuration of the Envoy component built into Cilium agents.

This feature is enabled using the `--enable-envoy-config` feature flag.

## Load Balancing example using Kind

This example shows an example configuration that 
* load-balances between two backend services
* performs a URL re-write
* shows L7 observability

### Deploy a Kind cluster with Cilium Service Mesh installed

The [Cilium CLI repository](https://github.com/cilium/cilium-cli) includes a configuration file for a two-node [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) cluster that we can use for this example. 

```
curl -o kind-config.yaml https://raw.githubusercontent.com/cilium/cilium-cli/master/.github/kind-config.yaml
kind cluster create --config kind-config.yaml
```

Install Cilium using the images built for the Service Mesh beta, and with the feature flag on:

```
cilium install --version :<SERVICE_MESH_BETA> --relay-version :<SERVICE_MESH_BETA> --operator-image <OPERATOR_SERVICE_MESH_IMAGE>  --config debug=true --config debug-verbose=datapath --config enable-envoy-config=true --config monitor-aggregation=none --kube-proxy-replacement=probe 
cilium hubble enable
cilium hubble port-forward&
```

### Deploy test applications

Run `cilium connectivity test` to deploy the test applications used for L7 egress tests:

```
cilium connectivity test --test egress-l7
```

These test services consist of 
* two client deployments, `client` and `client2`
* two backend services, `echo-other-node` and `echo-same-node` 

View information about these pods:

```
kubectl get pods -n cilium-test --show-labels -o wide 
```

You can see that 
* Only `client2` is labelled with `other=client` - we will use this in a `CiliumNetworkPolicy` definition later in this example
* The pods for `client`, `client2` and `echo-same-node` run on one node, while `echo-other-node` is scheduled to the other node

Make a note of the client pod ID: 

```
export CLIENT=<client pod ID>
```

You can also view the `echo-` services:

```
kubectl get svc -n cilium-test
```

We are going to use Envoy configuration to load-balance requests between these two backend services `echo-other-node` and `echo-same-node`. 

### Start observing traffic with Hubble 

Start a second terminal, and observe traffic from the `client2` pod: 

```
hubble observe --from-pod cilium-test/$CLIENT -f
```

You should be able to get a response from both of the backend services individually from `client2`:

```
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-same-node:8080/
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-other-node:8080/
```

Notice that Hubble shows all the flows as being either `to/from-overlay` or `to/from-endpoint` - there is no traffic marked as flowing to or from the proxy at this stage.

Verify that you get a 404 error response if you curl to the non-existant URL `/foo` on these services: 

```
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-same-node:8080/foo
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-other-node:8080/foo
```

### Add Layer 7 policy

Adding Layer 7 policy introduces the Envoy proxy into the path for this traffic. 

```
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium-cli/master/connectivity/manifests/client-egress-l7-http.yaml
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium-cli/master/connectivity/manifests/client-egress-only-dns.yaml
```

Make a request to a backend service (either will do): 

```
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-same-node:8080/
```

Notice that the Hubble output now includes flows `to-proxy`, and also shows the HTTP protocol information at level 7 (for example `(HTTP/1.1 GET http://echo-same-node:8080/)`

The policy only permits GET requests to the `/` path, so you will see requests to any other URL being dropped. For example, try: 

```
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-same-node:8080/foo
```

### Add Envoy load-balancing and URL re-writing

Apply the `envoy-test.yaml` file, which defines a `CiliumEnvoyConfiguration`

```
kubectl apply -f envoy-test.yaml
```

This configuration listens for traffic intended for either of the two `echo-` services and 
* load-balances 50/50 between the two backend `echo-` services
* rewrites the path `/foo` to `/`

A request to `/foo` should now succeed, because of the path re-writing: 

```
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-same-node:8080/foo
```

But the network policy still prevents requests to any path that is not rewritten to `/`. For example, this request will result in a packet being dropped and a `403 Forbidden` response code: 

```
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-same-node:8080/bar
```

Try making several requests to the one backend service, and you should see in the Hubble output that half the time, they are handled by the other backend. 

Example:

```
Oct 13 16:30:59.023: cilium-test/client2-6dd75b74c6-68h7d:45004 <> cilium-test/echo-other-node:8080 from-endpoint FORWARDED (TCP Flags: SYN)
Oct 13 16:30:59.032: cilium-test/client2-6dd75b74c6-68h7d:45004 <> cilium-test/echo-other-node-697d5d69b7-x6qnp:8080 from-proxy FORWARDED (TCP Flags: SYN)
Oct 13 16:31:10.717: cilium-test/client2-6dd75b74c6-68h7d:45164 <> cilium-test/echo-other-node:8080 from-endpoint FORWARDED (TCP Flags: SYN)
Oct 13 16:31:10.721: cilium-test/client2-6dd75b74c6-68h7d:45164 <> cilium-test/echo-same-node-7967996674-t24mq:8080 from-proxy FORWARDED (TCP Flags: SYN)
```
