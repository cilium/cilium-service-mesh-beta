# Layer 7 Traffic Management

Cilium Service Mesh defines a `CiliumEnvoyConfig` CRD which allows users to set the configuration of the Envoy component built into Cilium agents.

This feature is enabled using the `--enable-envoy-config` feature flag.

**_Note: there is currently an [issue](https://github.com/cilium/cilium-service-mesh-beta/issues/9) with Envoy traffic not being subjected to datapath processing properly in direct datapath mode. The workaround is to run Cilium in tunnelling mode, by installing with `--datapath-mode=vxlan` as an option on `cilium install`_** 

This example sets up an Envoy listener which load balances requests between two backend services. 

## Deploy test applications

You will need a Kubernetes cluster with at least two nodes for this example. Start by [installing Cilium to your test cluster](../INSTALLATION.md).

Run `cilium connectivity test` to deploy the test applications used for L7 egress tests:

```
cilium connectivity test --test egress-l7
```

The test workloads run in the namespace `cilium-test` and consist of 
* two client deployments, `client` and `client2`
* two backend services, `echo-other-node` and `echo-same-node` 

View information about these pods:

```
kubectl get pods -n cilium-test --show-labels -o wide 
```

You can see that 
* Only `client2` is labelled with `other=client` - we will use this in a `CiliumNetworkPolicy` definition later in this example
* The pods for `client`, `client2` and `echo-same-node` run on one node, while `echo-other-node` is scheduled to another node

Make an environment variable with the pod ID for `client2`: 

```
export CLIENT2=<client pod ID>
```

We are going to use Envoy configuration to load-balance requests between these two backend services `echo-other-node` and `echo-same-node`. 

### Start observing traffic with Hubble 

Start a second terminal, and observe traffic from the `client2` pod: 

```
hubble observe --from-pod cilium-test/$CLIENT2 -f
```

You should be able to get a response from both of the backend services individually from `client2`:

```
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-same-node:8080/
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-other-node:8080/
```

Notice that Hubble shows all the flows between these pods as being either `to/from-stack`, `to/from-overlay` or `to/from-endpoint` - there is no traffic marked as flowing to or from the proxy at this stage. (This assumes you don't already have any Layer 7 policies in place affecting this traffic.)

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

Adding Layer 7 policy enables Layer 7 visibility. Notice that the Hubble output now includes flows `to-proxy`, and also shows the HTTP protocol information at level 7 (for example `(HTTP/1.1 GET http://echo-same-node:8080/)`

### Test Layer 7 policy enforcement

The policy only permits GET requests to the `/` path, so you will see requests to any other URL being dropped. For example, try: 

```
kubectl exec -it -n cilium-test $CLIENT2 -- curl -v echo-same-node:8080/foo
```

The Hubble output will show the HTTP request being dropped, like this: 

```
Dec 15 15:10:17.639: cilium-test/client2-5998d566b4-566p5:51312 -> cilium-test/echo-same-node-745bd5c77-xggqw:8080 http-request DROPPED (HTTP/1.1 GET http://echo-same-node:8080/foo)
```

And the curl shoud show a 403 Forbidden response. 

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

Try making several requests to one backend service, and you should see in the Hubble output that half the time, they are handled by the other backend. 

Example:

```
Dec 16 13:59:50.430: cilium-test/client2-5998d566b4-j6jms:37748 -> cilium-test/echo-same-node-745bd5c77-5xfmv:8080 http-request FORWARDED (HTTP/1.1 GET http://echo-same-node:8080/)
...
Dec 16 13:59:53.584: cilium-test/client2-5998d566b4-j6jms:37822 -> cilium-test/echo-other-node-f4d46f75b-l5cgp:8080 http-request FORWARDED (HTTP/1.1 GET http://echo-same-node:8080/)
...
Dec 16 14:00:13.122: cilium-test/client2-5998d566b4-j6jms:37982 -> cilium-test/echo-same-node-745bd5c77-5xfmv:8080 http-request FORWARDED (HTTP/1.1 GET http://echo-same-node:8080/)
...
Dec 16 14:00:21.959: cilium-test/client2-5998d566b4-j6jms:37822 -> cilium-test/echo-other-node-f4d46f75b-l5cgp:8080 http-request FORWARDED (HTTP/1.1 GET http://echo-same-node:8080/)
```
