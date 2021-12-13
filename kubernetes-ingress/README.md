# Kubernetes Ingress 

Cilium uses the standard Kubernetes Ingress resource definition, with an `ingressClassName` of `cilium`. This can be used for path-based routing and for TLS termination

*Note: that the ingress controller creates a service of LoadBalancer type, so [your environment will need to support this](https://github.com/cilium/cilium-service-mesh-beta/issues/3).*

Examples: 
* [HTTP](http.md)
* [TLS Termination]() **coming soon**
* [gRPC]() **coming soon**




