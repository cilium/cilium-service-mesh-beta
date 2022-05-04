# Installing Cilium Service Mesh

These instructions should help you get the Cilium Service Mesh-specific images installed in your cluster, with the service mesh features enabled. 

## Install Cilium and Hubble

[Install the Cilium CLI](https://docs.cilium.io/en/v1.11/gettingstarted/k8s-install-default/#install-the-cilium-cli) and run `cilium version` to check that you have version 0.10.2 or above.

Install Cilium with the service mesh builds, and enable Hubble:

```
cilium  install --version v1.12.0-rc1 --helm-set enableIngressController=true --kube-proxy-replacement=probe
cilium hubble enable
```

Alternatively, you can use Helm to install Cilium Service Mesh. For example, to install on GKE:

```
# Specify your cluster name and zone.
CLUSTER_NAME=my-cluster
CLUSTER_ZONE=us-west2-a

NATIVE_CIDR="$(gcloud container clusters describe $CLUSTER_NAME --zone $CLUSTER_ZONE --format 'value(clusterIpv4Cidr)')"
git clone git@github.com:cilium/cilium.git
cd cilium
git checkout origin/beta/service-mesh
cat > service-mesh-values.yaml <<EOF
image:
  repository: quay.io/cilium/cilium-service-mesh
  tag: v1.11.0-beta.1
  useDigest: false
operator:
  image:
    repository: quay.io/cilium/operator
    tag: v1.11.0-beta.1
    useDigest: false
    suffix: "-service-mesh"
kubeProxyReplacement: probe
hubble:
  relay:
    enabled: "true"
extraConfig:
  enable-envoy-config: "true"
# Following settings are for specific to GKE. Please adjust them according to
# https://docs.cilium.io/en/v1.11/gettingstarted/k8s-install-helm/#installation-using-helm
# if you are using other Kubernetes environments.
nodeinit:
  enabled: true
  reconfigureKubelet: true
  removeCbrBridge: true
cni:
  binPath: /home/kubernetes/bin
gke:
  enabled: true
ipam:
  mode: kubernetes
nativeRoutingCIDR: ${NATIVE_CIDR}
EOF
helm install -n kube-system cilium ./install/kubernetes/cilium -f service-mesh-values.yaml
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
Image versions    cilium             quay.io/cilium/cilium:v1.12.0-rc1: 2
                  cilium-operator    quay.io/cilium/operator-generic:v1.12.0-rc1: 1
                  hubble-relay       quay.io/cilium/hubble-relay:v1.12.0-rc1: 1
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
Image versions    cilium             quay.io/cilium/cilium:v1.12.0-rc1: 2
                  cilium-operator    quay.io/cilium/operator-generic:v1.12.0-rc1: 1
                  hubble-relay       quay.io/cilium/hubble-relay:v1.12.0-rc1: 1
                  hubble-ui          quay.io/cilium/hubble-ui:v0.8.5@sha256:4eaca1ec1741043cfba6066a165b3bf251590cf4ac66371c4f63fbed2224ebb4: 1
                  hubble-ui          quay.io/cilium/hubble-ui-backend:v0.8.5@sha256:2bce50cf6c32719d072706f7ceccad654bfa907b2745a496da99610776fe31ed: 1
                  hubble-ui          docker.io/envoyproxy/envoy:v1.20.2@sha256:eb7d88d5186648049f0a80062120bd45e7557bdff3f6a30e1fc92cbb50916868: 1
```

The following command will open the UI in your browser: 

```
cilium hubble ui
```
