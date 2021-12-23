# Ingress example with TLS termination 

This example builds on the [HTTP](http.md) and [gRPC](grpc.md) ingress examples, adding TLS
termination. 

## Create TLS certificate and private key

For demonstration purposes we will use a TLS certificate signed by a made-up, self-signed 
certificate authority (CA). One easy way to do this is with
[`minica`](https://github.com/jsha/minica). We want a certificate that will
validate bookinfo.cilium.rocks and hipstershop.cilium.rocks, as
these are the host names used in this ingress example.

```
minica -domains '*.cilium.rocks'
```

On first run, `minica` generates a CA certificate and key (`minica.pem` and
`minica-key.pem`). It also creates a directory called `_.cilium.rocks`
containing a key and certificate file that we will use for the ingress service.   

Create a Kubernetes secret with this demo key and certificate: 

```
kubectl create secret tls demo-cert --key=bookinfo.cilium.rocks/key.pem --cert=bookinfo.cilium.rocks/cert.pem 
```

## Deploy the ingress

If you previously ran the HTTP and/or gRPC ingress demos, delete the ingresses: 

```
kubectl delete ingress basic-ingress
kubectl delete ingress grpc-ingress
```

The ingress configuration for this demo provides the same routing as those demos
but with the addition of TLS termination. Deploy this ingress: 

```
kubectl apply -f tls-ingress.yaml 
```

This creates a LoadBalancer service, which after around 30 seconds or so should
be populated with an external IP address. 

```
❯ kubectl get ingress                          
NAME          CLASS    HOSTS                                            ADDRESS        PORTS     AGE
tls-ingress   cilium   hipstershop.cilium.rocks,bookinfo.cilium.rocks   35.195.24.75   80, 443   6m5s
```

## Edit /etc/hosts

In this ingress configuration, the host names `hipstershop.cilium.rocks` and
`bookinfo.cilium.rocks` are specified in the path routing rules. The client
needs to specify which host it wants to access. This can be achieved by
editing your local `/etc/hosts` file. (You will almost certainly need to be
superuser to edit this file.) Add entries using the IP address
assigned to the ingress service, so your file looks something like this: 

```
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

```
curl --cacert minica.pem -v https://bookinfo.cilium.rocks/details/1
```

If you prefer, instead of supplying the CA you can specify `-k` to tell the curl client not to validate the
server's certificate. Without either, you will get an error that the certificate
was signed by an unknown authority. 

Specifying -v on the curl request, you can see that the TLS handshake took place successfully. 

Similarly you can specify the CA on a gRPC request like this:

```
grpcurl -proto ./demo.proto -cacert ca-cert.pem hipstershop.cilium.rocks:443 hipstershop.ProductCatalogService/ListProducts
```

(See the [gRPC](grpc.md) example if you don't already have the `demo.proto` file downloaded.)

You can also visit https://bookinfo.cilium.rocks in your browser. The browser
will warn you that the certificate authority is unknown but if you proceed past
this, you should see the bookstore application home page. 

Note that requests will time out if you don't specify `https://`. 
