# cert-manager-demo

A demo of securing an Kubernetes ingress resource using self signed certificate powered by cert-manager. `cert-manager` will automatically provision and manage certificate in Kubernetes.

1. Install cert-manager using helm chart
```bash
$ helm repo add jetstack https://charts.jetstack.io

$ helm repo update

$ helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.0 \
  --set installCRDs=true

```

2. Create a ca.key and ca.crt for Certificate Issuer
```bash
$ openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out ca.crt -keyout ca.key

```

3. Create a Kubernetes TLS secret containing key and crt file of the Issuer
```bash
$ kubectl create secret tls ca-key-pair --key=ca.key --cert=ca.crt

```

4. Create a Issuer(CR) with ca-key-pair secret
```bash
$ kubectl apply -f issuer.yaml

```

5. Issue a Certificate(CR) for the ingress resource
```bash
$ kubectl apply -f certificate.yaml

```

6. Create a deployment
```bash
$ kubectl create deployment nginx --image=nginx

```

7. Expose the deployment using a service
```bash
$ kubectl expose deployment nginx --port 8080 --target-port 80

```

8. Deploy an ingress resource for the nginx service. Ingress will expose the service at `example.local`
```bash
$ kubectl apply -f ingress.yaml

```

9. Access TLS secured ingress using curl
```bash
$ curl --cacert ca.crt https://example.local

```

Note: https://example.local can also be accessed from the browser but it will give an alert. Because of our self-signed issuer, browser can't recognize the CA.

Resource: [cert-manager doc](https://cert-manager.io/docs/)
