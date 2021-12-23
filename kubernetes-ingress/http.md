# Ingress HTTP example

The example ingress configuration routes traffic to backend services from the `bookinfo` demo microservices app from the Istio project. 

Deploy the demo app: 

```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/bookinfo/platform/kube/bookinfo.yaml
```

*Note: this is just deploying the demo app, it's not adding any Istio components*

## Deploy the ingress

You'll find the example Ingress definition in `basic-ingress.yaml`.

```
kubectl apply -f basic-ingress.yaml 
```

This example routes requests for the path `/details` to the `details` service, and `/` to the `productpage` service. 

Getting the list of services, you'll see a LoadBalancer service is automatically created for this ingress. Your cloud provider will automatically provision an external IP address, but it may take around 30 seconds.

```
❯ kubectl get svc
NAME                           TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)        AGE
cilium-ingress-basic-ingress   LoadBalancer   10.24.11.148   <IP address>      80:30543/TCP   19d
details                        ClusterIP      10.24.7.97     <none>            9080/TCP       19d
kubernetes                     ClusterIP      10.24.0.1      <none>            443/TCP        19d
productpage                    ClusterIP      10.24.2.38     <none>            9080/TCP       19d
ratings                        ClusterIP      10.24.11.9     <none>            9080/TCP       19d
reviews                        ClusterIP      10.24.13.46    <none>            9080/TCP       19d
```

The external IP address should also be populated into the Ingress: 

```
❯ kubectl get ingress
NAME            CLASS    HOSTS   ADDRESS           PORTS   AGE
basic-ingress   cilium   *       <IP address>      80      19d
```

*Note: some providers e.g. EKS use a fully-qualified domain name rather than an IP address. In this case, for now you will need to get the domain name from the service as it won't be populated into the Ingress. Connectivity shouldn't be affected. 

## Make HTTP requests

Check (with `curl` or in your browser) that you can make HTTP requests to that external address. The `/` path takes you to the home page for the bookinfo application. 

From outside the cluster you can also make requests directly to the `details` service using the path `/details`. But you can't directly access other URL paths that weren't defined in `basic-ingress.yaml`. For example, you can get JSON data from a request to  `<address>/details/1` and get back some data, but you will get a 404 error if you make a request to `<address>/ratings`. 

