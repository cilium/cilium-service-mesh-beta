# Ingress gRPC example

The example ingress configuration in `grpc-ingress.yaml` shows how to route gRPC traffic to backend
services.

## Install grpcurl 

To issue client gRPC requests you can use
[grpcurl](https://github.com/fullstorydev/grpcurl#binaries). 

## Deploy the demo app

For this demo we will use [GCP's microservices demo app](https://github.com/GoogleCloudPlatform/microservices-demo). Install the app: 

```
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml
```
Since gRPC is binary-encoded, you also need the proto definitions for the gRPC
services in order to make gRPC requests. Download this for the demo app: 

```
curl -o demo.proto https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/pb/demo.proto
```

## Deploy the ingress

You'll find the example Ingress definition in `grpc-ingress.yaml`.

```
kubectl apply -g grpc-ingress.yaml
```

This defines paths for requests to be routed to the `productcatalogservice` and
`currencyservice` microservices. 

Just as in the [HTTP ingress demo](http.md) this creates a LoadBalancer
service, and it may take a little while for your cloud provider to provision an
external IP address. 

```
❯ kubectl get ingress
NAME            CLASS    HOSTS   ADDRESS           PORTS   AGE
grpc-ingress    cilium   *       <IP address>      80      3d
```

Store that IP address in an environment variable `INGRESS_IP`.

## Make gRPC requests to backend services 

To access the currency service:

```
grpcurl -plaintext -proto ./demo.proto $INGRESS_IP:80 hipstershop.CurrencyService/GetSupportedCurrencies
```

To access the product catalog service: 

```
grpcurl -plaintext -proto ./demo.proto $INGRESS_IP:80 hipstershop.ProductCatalogService/ListProducts
```




