# Installing Cilium Service Mesh

These instructions should help you get the Cilium Service Mesh-specific images installed in your cluster, with the service mesh features enabled. 

## Install Cilium

[Install the Cilium CLI](https://docs.cilium.io/en/v1.11/gettingstarted/k8s-install-default/#install-the-cilium-cli) and run `cilium version` to check that you have version 0.10.2 or above.

Install Cilium with the service mesh builds. 

* If you want Cilium to co-exist alongside kube-proxy:

```
cilium install --version -service-mesh:v1.11.0-beta.1 --config enable-envoy-config=true --kube-proxy-replacement=partial --config enable-node-port=true
```

* If you want to Cilium to completely replace kube-proxy, you will also need to specify the IP address of the node in which the Kubernetes API Server is running: 

```
cilium install --version -service-mesh:v1.11.0-beta.1 --config enable-envoy-config=true --kube-proxy-replacement=strict --config k8s-api-server=<API server URL>
```

Note that Cilium Service Mesh will not work with `--kube-proxy-replacement=disabled`. 

## Install Hubble 

Enable Hubble:

```
cilium hubble enable
```

## Confirm installation 

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

### Problems? 

If you see errors during installation, it would be helpful you can try uninstalling, then re-installing the regular release of Cilium to see whether it's a problem specific to the service mesh builds. 

```
cilium uninstall 
cilium install 
cilium hubble enable
```

## Use Hubble 

In order to use the `hubble` command to observe flows you will want to use port-forwarding: 

```
cilium hubble port-forward & 
```

[Install the Hubble CLI](https://docs.cilium.io/en/v1.11/gettingstarted/hubble_setup/#install-the-hubble-client), and then you can continuously observe flows in the cluster with commands such as: 

```
hubble observe -f
```

Check out `hubble help observe` for command line options to filter the output down to what you are most interested in (e.g., `--to-service`).

## (Optional) Install Hubble UI

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
