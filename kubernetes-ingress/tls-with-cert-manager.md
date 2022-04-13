# Ingress example with TLS termination using cert-manager

This example builds on the [HTTP](http.md) and [gRPC](grpc.md) ingress
examples, adding TLS termination. This example is similar to the
[TLS](tls.md) ingress example, in which you had to manually create a
self-signed CA using `minica`. In this example, you will be using
[cert-manager](https://cert-manager.io/), a Kubernetes-native operator that
issues and renews TLS certificates for you.

## Create TLS certificate and private key

Let us install cert-manager:

```sh
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager --version v1.7.1 --namespace cert-manager --set installCRDs=true --create-namespace
```

Now, create a CA Issuer:

```sh
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium-service-mesh-beta/main/kubernetes-ingress/ca-issuer.yaml
```

## Deploy the ingress

Follow the instructions from the [TLS demo](TLS.md) to deploy the ingress.

These instructions will get you to install a LoadBalancer service, which after
around 30 seconds or so should be populated with an external IP address.

```console
$ kubectl get ingress
NAME          CLASS    HOSTS                                            ADDRESS        PORTS     AGE
tls-ingress   cilium   hipstershop.cilium.rocks,bookinfo.cilium.rocks   35.195.24.75   80, 443   6m5s
```

To tell cert-manager that this Ingress needs a certificate, annotate the
Ingress with the name of the CA issuer we previously created:

```sh
kubectl annotate ingress tls-ingress cert-manager.io/issuer=ca-issuer
```

This creates a Certificate object along with a Secret containing the TLS
certificate.

```console
$ kubectl get certificate,secret demo-cert
NAME                                    READY   SECRET      AGE
certificate.cert-manager.io/demo-cert   True    demo-cert   33m

NAME               TYPE                DATA   AGE
secret/demo-cert   kubernetes.io/tls   3      33m
```

_Note: if you previously followed the the basic [TLS example](TLS.md) there was a pre-existing secret called `demo-cert`. This will be properly "updated" by cert-manager but the updated secret won't be picked up by Cilium ([#22](https://github.com/cilium/cilium-service-mesh-beta/issues/22)). You can work around this by deleting and re-creating the service `cilium-ingress-tls-ingress` after checking that cert-manager has properly created the secret. One way to do this is to delete and re-create the ingress config file tls-ingress.yaml_

## Edit /etc/hosts

Follow the same instructions as in the basic [TLS demo](TLS.md) to edit your
`/etc/hosts` file with the IP address assigned to the ingress.

## Make requests

By specifying the CA's certificate on a curl request, you can say that you trust
certificates signed by that CA.

This is very similar to the basic TLS demo, except that the CA's secret needs to
be retrieved from the Kubernetes secret where it's stored.

```sh
curl --cacert <(kubectl get secret ca -o="jsonpath={.data.ca\.crt}" | base64 -d) \
    -v https://bookinfo.cilium.rocks/details/1
```

If you prefer, instead of supplying the CA you can specify `-k` to tell the curl
client not to validate the server's certificate. Without either, you will get an
error that the certificate was signed by an unknown authority.

Specifying `-v` on the curl request, you can see that the TLS handshake took
place successfully.

Similarly you can specify the CA on a gRPC request like this:

```sh
grpcurl -proto ./demo.proto -cacert <(kubectl get secret ca -o="jsonpath={.data.ca\.crt}" | base64 -d) \
    hipstershop.cilium.rocks:443 hipstershop.ProductCatalogService/ListProducts
```

(See the [gRPC](grpc.md) example if you don't already have the `demo.proto` file downloaded.)

You can also visit <https://bookinfo.cilium.rocks> in your browser. The browser
will warn you that the certificate authority is unknown but if you proceed past
this, you should see the bookstore application home page.

Note that requests will time out if you don't specify `https://`.
