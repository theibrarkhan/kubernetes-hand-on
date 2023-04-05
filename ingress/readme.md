# Kubernetes Ingress with self sign tls

## simple nginx deployment

```
kubectl apply -f ingress-deployment.yaml

```

## create cert and key

```
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.crt -subj "/CN=abkhan.com" -days 365

abkhan.com would be your domain name

```

## create secret via cli

```
kubectl create secret tls abkhan-com-tls --cert=tls.crt --key=tls.key

```

## Add server ip map to the domain in /etc/hosts file

like 10.0.0.20 hello.com

## Ingress

```
kubectl apply -f ingress.yaml

```
