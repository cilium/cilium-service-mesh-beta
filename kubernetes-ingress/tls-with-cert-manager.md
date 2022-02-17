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

If you previously ran the HTTP and/or gRPC ingress demos, delete the ingresses:

```sh
kubectl delete ingress basic-ingress grpc-ingress
```

The ingress configuration for this demo provides the same routing as those demos
but with the addition of TLS termination. Deploy this ingress:

```sh
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium-service-mesh-beta/main/kubernetes-ingress/tls-ingress.yaml
```

This creates a LoadBalancer service, which after around 30 seconds or so should
be populated with an external IP address.

```sh
❯ kubectl get ingress
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

```sh
$ kubectl get certificate,secret demo-cert
NAME                                    READY   SECRET      AGE
certificate.cert-manager.io/demo-cert   True    demo-cert   33m

NAME               TYPE                DATA   AGE
secret/demo-cert   kubernetes.io/tls   3      33m
```

## Edit /etc/hosts

In this ingress configuration, the host names `hipstershop.cilium.rocks` and
`bookinfo.cilium.rocks` are specified in the path routing rules. The client
needs to specify which host it wants to access. This can be achieved by
editing your local `/etc/hosts` file. (You will almost certainly need to be
superuser to edit this file.) Add entries using the IP address
assigned to the ingress service, so your file looks something like this:

```sh
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
...
34.79.55.0 bookinfo.cilium.rocks
34.79.55.0 hipstershop.cilium.rocks
```

## Make requests

By specifying the CA's certificate on a curl request, you can say that you trust certificates
signed by that CA.

```sh
curl --cacert <(kubectl get secret ca -o="jsonpath={.data.ca\.crt}" | base64 -d) \
    -v https://bookinfo.cilium.rocks/details/1
```

If you prefer, instead of supplying the CA you can specify `-k` to tell the curl client not to validate the
server's certificate. Without either, you will get an error that the certificate
was signed by an unknown authority.

Specifying -v on the curl request, you can see that the TLS handshake took place successfully.

Similarly you can specify the CA on a gRPC request like this:

```sh
grpcurl -proto ./demo.proto -cacert <(kubectl get secret ca -o="jsonpath={.data.ca\.crt}" | base64 -d) \
    hipstershop.cilium.rocks:443 hipstershop.ProductCatalogService/ListProducts
```

(See the [gRPC](grpc.md) example if you don't already have the `demo.proto` file downloaded.)

You can also visit https://bookinfo.cilium.rocks in your browser. The browser
will warn you that the certificate authority is unknown but if you proceed past
this, you should see the bookstore application home page.

Note that requests will time out if you don't specify `https://`.
