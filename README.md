# DRAFT

To do before release 

- [ ] Finish instructions for K8s Ingress
- [ ] Finish instructions for L7 traffic / CEC
- [ ] Update Cilium component image names (and add a Releases section to this repo?) 
- [ ] Update Cilium CLI version TBD
- [ ] Make this repo public 
- [ ] Check that Issue Templating works (it's not supported in private repos) 

# Cilium Service Mesh beta program

Welcome to the Cilium Service Mesh beta program! 

The Cilium project is developing a set of Service Mesh features and capabilities. We're doing this in a feature branch separate from the production release of Cilium, to give us the flexibility to make potentially non-backwards-compatible changes, for example changes to CRDs. Once Service Mesh features are stable, they will be merged into the [main Cilium repo](https://github.com/cilium/cilium). (Read more about the [motivations for the beta program](https://cilium.io/blog/2021/12/01/cilium-service-mesh-beta)). 

In this repo you'll find 
* information about the current status of Cilium Service Mesh features
* instructions for getting started with Cilium Service Mesh
* a [discussions forum](https://github.com/cilium/cilium-service-mesh-beta/discussions) for your feedback on Cilium Service Mesh features and usability
* an [issue tracker](https://github.com/cilium/cilium-service-mesh-beta/issues) for bug reports related to Cilium Service Mesh. 

## Service Mesh feature status 

This is the readiness of Service Mesh capabilities in the Service Mesh feature branch, and built into the latest set of service mesh beta images. Please note: although we’ll only change things for good reasons, there are no backward-compatibility guarantees with the configuration of Cilium Service Mesh during beta, and the code has not only been through limited testing. Please be mindful of this when you choose where to deploy Cilium Service Mesh - we do not recommend using it in production or staging environments yet.

**Alpha** Very early release of features that have not been through extensive testing yet. No guarantee that future releases will be back compatible with CRDs or other configuration settings. We would love your feedback on Alpha features, but please don't be surprised if you encounter bugs.

**Beta** Early release of features that we expect to be stable enough for testing by end users. No guarantee that future releases will be back compatible with CRDs or other configuration settings. We would love your feedback and issue reports about Beta features. 

**v1.11** Features already merged into the Cilium v1.11 release (as well as being available in the Service Mesh beta-specific builds)

| Feature | Status | 
|---------|--------|
| Kubernetes Ingress support | Beta |
| Open Telemetry support | Alpha, v1.11 |
| L7-aware Traffic Management | Alpha | 

Other features will be added as the beta progresses. 

## Image tags

Container images built to include the service mesh beta features are available from `quay.io`: 

| Component | Image name | 
|-----------|------------|
| Cilium agent | quay.io/cilium/cilium-service-mesh:v1.11.0-beta.1 |
| Cilium operator | quay.io/cilium/operator-generic-service-mesh:v1.11.0-beta.1 | 
| Hubble relay | quay.io/cilium/operator-generic-service-mesh:v1.11,0-beta.1 | 

You will also need [Cilium CLI](https://github.com/cilium/cilium-cli). Check that you have version 0.9.4 or greater installed by running `cilium version` (or your own version built from the main branch of that repo). 

## Getting started

These instructions will help you test out Service Mesh features. 

### Kubernetes Ingress

Cilium Service Mesh adds support for Kubernetes Ingress with path-based routing for HTTP and gRPC traffic, and support for TLS termination. 

We have an example configuration and demo instructions for [Kubernetes Ingress](https://github.com/cilium/cilium-service-mesh-beta/tree/main/kubernetes-ingress) 

### Open Telemetry Support 

The Hubble observability tool now has an integration with Open Telemetry.  We have an example configuration and guide for trying out [Open Telemetry support](https://github.com/cilium/hubble-otel/blob/main/USER_GUIDE_KIND.md). 

### L7-aware Traffic Management

Cilium is deployed in Kubernetes as a DaemonSet, such that each node in the the cluster runs an instance of Cilium. This includes an Envoy container that, before Service Mesh, was already used for Layer 7 policy management and observability. In Cilium Service Mesh, a new `CiliumEnvoyConfiguration` CRD allows direct access to configuring Envoy L7 traffic management's capabilities. 

We have an example configuration and demo instructions for [L7-aware Traffic Management](https://github.com/cilium/cilium-service-mesh-beta/tree/main/l7-traffic-management).  

## Raising issues

If you find an issue related to Service Mesh please [raise it here](https://github.com/cilium/cilium-service-mesh-beta/issues). 

---

This beta program is part of the Cilium project and follows the same [Code of Conduct](https://github.com/cilium/cilium/blob/master/CODE_OF_CONDUCT.md). 
