# Ingress example with TLS termination 

This example builds on the [HTTP](http.md) and [gRPC](grpc.md) ingress examples, adding TLS
termination. 

## Create TLS certificate and private key

For demonstration purposes we will use a TLS certificate signed by a made-up 
certificate authority. First generate a key and self-signed certificate for this
certificate authority:

```
openssl genrsa 4096 > ca-key.pem
openssl req -new -x509 -nodes -key ca-key.pem -out ca-cert.pem 
```

You'll be asked a series of questions, but for the purposes of the demo feel
free to use any answers you like.

Generate a certificate request for the service. You can use the config file
`ssl.conf` from this directory for convenience.

```
openssl req -newkey rsa:4096 -nodes -keyout demo-key.pem -out demo-req.pem -config ssl.conf
```

Finally you can use this request to generate a certificate signed by the
(made-up) certificate authority:

```
openssl x509 -req -in demo-req.pem -out demo-cert.pem -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -extensions req_ext -extfile kubernetes-ingress/ssl.conf
```

Create a Kubernetes secret with this demo key and certificate: 

```
kubectl create secret tls demo-cert --key=demo-key.pem --cert=demo-cert.pem 
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
curl --cacert ca-cert.pem -v https://bookinfo.cilium.rocks/details/1
```

If you prefer, instead of supplying the CA you can specify `-k` to tell the curl client not to validate the
server's certificate. Without either, you will get an error that the certificate
was signed by an unknown authority. 

Specifying -v you can see that the TLS handshake took place successfully. 

Similarly you can specify the CA on a gRPC request like this:

```
grpcurl -proto ./demo.proto -cacert ca-cert.pem hipstershop.cilium.rocks:443 hipstershop.ProductCatalogService/ListProducts
```


