# DRAFT

To do before release 

- [ ] Add instructions for K8s Ingress
- [ ] Add instructions for L7 traffic / CEC
- [ ] Cilium component image builds
- [ ] Make this repo public 
- [ ] Check that Issue Templating works (it's not supported in private repos) 
- [ ] What else? 

# Cilium Service Mesh beta program

Welcome to the Cilium Service Mesh beta program! 

The Cilium project is developing a set of Service Mesh features and capabilities. We're doing this in a feature branch separate from the production release of Cilium, to give us the flexibility to make potentially non-backwards-compatible changes, for example changes to CRDs. Once Service Mesh features are stable, they will be merged into the [main Cilium repo](https://github.com/cilium/cilium). (Read more about the [motivations for the beta program](https://cilium.io/blog/2021/12/01/cilium-service-mesh-beta)). 

In this repo you'll find 
* information about the current status of Cilium Service Mesh features
* instructions for getting started with Cilium Service Mesh
* an issue tracker specifically for bug reports, usability concerns, and feature requests related to Cilium Service Mesh. 

## Service Mesh feature status 

This is the readiness of Service Mesh capabilities in the Service Mesh feature branch.

**Alpha** Very early release of features that have not been through extensive testing yet. No guarantee that future releases will be back compatible with CRDs or other configuration settings. We would love your feedback on Alpha features, but please don't be surprised if you encounter bugs.

**Beta** Early release of features that we expect to be stable enough for testing by end users. No guarantee that future releases will be back compatible with CRDs or other configuration settings. We would love your feedback and issue reports about Beta features. 

**v1.11** This feature is already merged into the Cilium v1.11 release (as well as being available in the Service Mesh beta-specific builds)

| Feature | Status | 
|---------|--------|
| Kubernetes Ingress support | Beta |
| Open Telemetry support | Alpha, v1.11 |
| L7-aware Traffic Management | Alpha | 

Other features will be added as the beta progresses. Get in touch on [Cilium Slack](http://slack.cilium.io) to let us know about your priorities! 

## Getting started

These instructions will help you test out Service Mesh features. 

### Kubernetes Ingress

Cilium Service Mesh adds support for Kubernetes Ingress with path-based routing for HTTP and gRPC traffic. 

[Kubernetes Ingress instructions]() To Be Added (based on Michi's internal demo set-up) 

### Open Telemetry Support 

The Hubble observability tool now has an integration with Open Telemetry.  We have a guide for trying out [Open Telemetry support](https://github.com/cilium/hubble-otel/blob/main/USER_GUIDE_KIND.md) in a Kind cluster. 

### L7-aware Traffic Management

Cilium is deployed in Kubernetes as a DaemonSet, such that each node in the the cluster runs an instance of Cilium. This includes an Envoy container that, before Service Mesh, was already used for Layer 7 policy management and observability. In Cilium Service Mesh, a new `CiliumEnvoyConfiguration` CRD allows direct access to configuring Envoy L7 traffic management's capabilities. 

[L7-aware Traffic Management]() To Be Added (based on Jarno's PR README) 

## Raising issues

If you find an issue related to Service Mesh please [raise it here](https://github.com/cilium/cilium-service-mesh-beta/issues). 

---

This beta program is part of the Cilium project and follows the same [Code of Conduct](https://github.com/cilium/cilium/blob/master/CODE_OF_CONDUCT.md). 
