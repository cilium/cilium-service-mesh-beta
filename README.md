# Cilium Service Mesh beta program

Welcome to the Cilium Service Mesh beta program! 

In this repo you'll find 
* information about the current status of Cilium Service Mesh features
* **Coming soon**: instructions for getting started with Cilium Service Mesh 
* a [discussions forum](https://github.com/cilium/cilium-service-mesh-beta/discussions) for your feedback on Cilium Service Mesh features and usability
* an [issue tracker](https://github.com/cilium/cilium-service-mesh-beta/issues) for bug reports related to Cilium Service Mesh. 

The Cilium project is developing Service Mesh capabilities in a [feature branch](https://github.com/cilium/cilium/tree/beta/service-mesh), to give us the flexibility to make potentially non-backwards-compatible changes, for example changes to CRDs. We plan to merge Service Mesh features into Cilium release v1.12 in early 2022. (Read more about the [motivations for the beta program](https://cilium.io/blog/2021/12/01/cilium-service-mesh-beta)). 

## Service Mesh feature status 

*Please note: although we’ll only change things for good reasons, there are no backward-compatibility guarantees with the configuration of Cilium Service Mesh during beta, and the code has only been through limited testing. Please be mindful of this when you choose where to deploy Cilium Service Mesh - we do not recommend using it in production or staging environments yet.*

**Alpha** Very early release of features that have not been through extensive testing yet. No guarantee that future releases will be back compatible with CRDs or other configuration settings. Please don't be surprised if you encounter bugs.

**Beta** Early release of features that we expect to be stable enough for testing by end users. No guarantee that future releases will be back compatible with CRDs or other configuration settings.

**v1.11** Features already merged into the Cilium v1.11 release (as well as in the Service Mesh beta-specific builds)

| Feature | Status | 
|---------|--------|
| Kubernetes Ingress support | Beta |
| Open Telemetry support | Alpha, v1.11 |
| L7-aware Traffic Management | Alpha | 

Other features will be added as the beta progresses. 

## Image tags

Container images built to include the service mesh beta features: 

| Component | Image name | 
|-----------|------------|
| Cilium agent | quay.io/cilium/cilium-service-mesh:v1.11.0-beta.1 |
| Cilium operator | quay.io/cilium/operator-generic-service-mesh:v1.11.0-beta.1 | 
| Hubble relay | quay.io/cilium/operator-generic-service-mesh:v1.11,0-beta.1 | 

You will also need [Cilium CLI](https://github.com/cilium/cilium-cli) version 0.9.4 or greater. (Check the version with `cilium version`). 

## Getting started

Instructions coming soon! 

## Raising issues

If you find an issue related to Service Mesh please [raise it here](https://github.com/cilium/cilium-service-mesh-beta/issues). 

---

This beta program is part of the Cilium project and follows the same [Code of Conduct](https://github.com/cilium/cilium/blob/master/CODE_OF_CONDUCT.md). 
