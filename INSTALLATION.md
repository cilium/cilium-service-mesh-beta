# Installing Cilium Service Mesh

These instructions should help you get the Cilium Service Mesh-specific images installed in your cluster, with the service mesh features enabled. 

## Install Cilium Service Mesh 

[Install the Cilium CLI](https://docs.cilium.io/en/v1.11/gettingstarted/k8s-install-default/#install-the-cilium-cli) and run `cilium version` to check that you have version 0.10.0 or above. 

Install Cilium with the service mesh builds, and enable Hubble:

```
cilium install --version -service-mesh:v1.11.0-beta.1 --config enable-envoy-config=true --kube-proxy-replacement=probe
cilium hubble enable
```

Check that Cilium is running correctly by running `cilium status`. You should see output like this. 

```
❯ cilium status
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         OK
 \__/¯¯\__/    ClusterMesh:    disabled
    \__/

Deployment        hubble-relay       Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet         cilium             Desired: 2, Ready: 2/2, Available: 2/2
Deployment        cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
Containers:       cilium             Running: 2
                  cilium-operator    Running: 1
                  hubble-relay       Running: 1
Cluster Pods:     3/3 managed by Cilium
Image versions    cilium-operator    quay.io/cilium/operator-aws-service-mesh:v1.11.0-beta.1: 1
                  hubble-relay       quay.io/cilium/hubble-relay-service-mesh:v1.11.0-beta.1: 1
                  cilium             quay.io/cilium/cilium-service-mesh:v1.11.0-beta.1: 2
```

Confirm that the image versions for cilium operator, hubble-relay, and cilium are the [latest beta builds](https://github.com/cilium/cilium-service-mesh-beta#image-tags).

In order to use the `hubble` command to observe flows you will want to use port-forwarding: 

```
cilium hubble port-forward & 
```

[Install the Hubble CLI](https://docs.cilium.io/en/v1.11/gettingstarted/hubble_setup/#install-the-hubble-client), and then you can observe flows in the cluster with commands such as: 

```
hubble observe --server localhost:4245
```

### (Optional) Install Hubble UI

If you'd like to visualize flows in the Hubble UI you'll need to install the hubble-ui components: 

```
cilium hubble enable --ui 
```

The images installed will be the standard images (there are no beta-specific builds for the UI components). 

```
❯ cilium status
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         OK
 \__/¯¯\__/    ClusterMesh:    disabled
    \__/

DaemonSet         cilium             Desired: 2, Ready: 2/2, Available: 2/2
Deployment        cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
Deployment        hubble-relay       Desired: 1, Ready: 1/1, Available: 1/1
Deployment        hubble-ui          Desired: 1, Ready: 1/1, Available: 1/1
Containers:       cilium             Running: 2
                  cilium-operator    Running: 1
                  hubble-relay       Running: 1
                  hubble-ui          Running: 1
Cluster Pods:     4/4 managed by Cilium
Image versions    cilium             quay.io/cilium/cilium-service-mesh:v1.11.0-beta.1: 2
                  cilium-operator    quay.io/cilium/operator-aws-service-mesh:v1.11.0-beta.1: 1
                  hubble-relay       quay.io/cilium/hubble-relay-service-mesh:v1.11.0-beta.1: 1
                  hubble-ui          docker.io/envoyproxy/envoy:v1.18.2@sha256:e8b37c1d75787dd1e712ff389b0d37337dc8a174a63bed9c34ba73359dc67da7: 1
                  hubble-ui          quay.io/cilium/hubble-ui:v0.8.3: 1
                  hubble-ui          quay.io/cilium/hubble-ui-backend:v0.8.3: 1
```

The following command will open the UI in your browser: 

```
cilium hubble ui
```
