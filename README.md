# Cilium Service Mesh beta program

Welcome to the Cilium Service Mesh beta program! 

In this repo you'll find 
* information about the [status of Cilium Service Mesh features](#service-mesh-feature-status) 
* [instructions for getting started](#getting-started) with Cilium Service Mesh 
* an [issue tracker](https://github.com/cilium/cilium-service-mesh-beta/issues) for bug reports related to Cilium Service Mesh. 

The Cilium project originally developed Service Mesh capabilities in a [feature branch](https://github.com/cilium/cilium/tree/beta/service-mesh), to give us the flexibility to make potentially non-backwards-compatible changes, for example changes to CRDs. Much of this functionality is now being merged into the main Cilium release ahead of v1.12. (Read more about the [motivations for the beta program](https://cilium.io/blog/2021/12/01/cilium-service-mesh-beta)). 

## Service Mesh feature status 

*Please note: although we’ll only change things for good reasons, there are no backward-compatibility guarantees with the configuration of Cilium Service Mesh during beta, and the code has only been through limited testing. Please be mindful of this when you choose where to deploy Cilium Service Mesh - we do not recommend using it in production or staging environments yet.*

**Alpha** Very early release of features that have not been through extensive testing yet. No guarantee that future releases will be back compatible with CRDs or other configuration settings. Please don't be surprised if you encounter bugs.

**Beta** Early release of features that we expect to be stable enough for testing by end users. No guarantee that future releases will be back compatible with CRDs or other configuration settings.

**v1.11** Features already merged into the Cilium v1.11 release (as well as in the Service Mesh beta-specific builds)

**v1.12** Features expected to be merged into the Cilium v.12 release

| Feature | Status | 
|---------|--------|
| Kubernetes Ingress support | Beta, v1.12 |
| Open Telemetry support | Alpha, v1.11, v1.12 |
| L7-aware Traffic Management | Alpha, v1.12 | 

Expected future features for Cilium Service Mesh are now listed in the [Cilium Roadmap](https://docs.cilium.io/en/latest/community/roadmap/#cilium-service-mesh)

### Known limitations

* People have reported problems [installing Cilium Service Mesh on AKS](https://github.com/cilium/cilium-service-mesh-beta/issues/18)

## Image tags

Container images built to include the service mesh beta features: 

| Component | Image name | 
|-----------|------------|
| Cilium agent | quay.io/cilium/cilium:v1.12.0-rc1 |
| Cilium operator | quay.io/cilium/operator-generic:v1.12.0-rc1 | 
| Hubble relay | quay.io/cilium/hubble-relay:v1.12.0-rc1 | 

You will also need [Cilium CLI](https://github.com/cilium/cilium-cli) version 0.10.0 or greater. (Check the version with `cilium version`). 

## Getting started

Instructions for installing the Cilium service mesh beta components are [here](INSTALLATION.md).

We have some example configurations and walkthroughs for trying out Service Mesh capabilities: 

* Try path-based routing and TLS termination with the [Kubernetes Ingress examples](./kubernetes-ingress/)
* Explore Cilium Envoy Configuration with examples showing [path translation](./custom-envoy-listener) and [Layer 7 load balancing](./l7-traffic-management/)
* Walk through an example of the [Hubble integration with OpenTelemetry](https://github.com/cilium/hubble-otel/blob/main/USER_GUIDE_KIND.md)
 
We'll be adding more during the course of the beta program (contributions welcome!) 

## Raising issues

If you find an issue related to Service Mesh please [raise it here](https://github.com/cilium/cilium-service-mesh-beta/issues). 

---

This beta program is part of the Cilium project and follows the same [Code of Conduct](https://github.com/cilium/cilium/blob/master/CODE_OF_CONDUCT.md). 
