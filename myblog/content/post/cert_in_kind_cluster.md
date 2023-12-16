---
title: "View certificate with ingress in kind cluster "
date: 2023-12-16T20:57:56+08:00
draft: false
tags: ["k8s"]
categories: ["k8s"]
author: "大白猫"
---

## Env

* Runtime: Colima
* Cluster: Kind
* Ingress controller: Contour
* Certification: Cert-manager

## Statement

I'm testing in my local env totally, and have no DNS. I want to check the self-signed cert created by issuer.

Already created a cluster with kind: [Contour_Create_a_Kind_Cluster](https://projectcontour.io/docs/1.27/guides/kind/)

Service: 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-name
  namespace: ns
spec:
  ports:
    - name: external
      port: 8080
      targetPort: 8080
  selector:
    app: app-name
  type: LoadBalancer
```

A self-signed issuer:

``` yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
  namespace: ns
spec:
  selfSigned: {}
```

Ingress:

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-name
  namespace: ns
  annotations:
    cert-manager.io/issuer: selfsigned-issuer
    cert-manager.io/common-name: ose.test.s3.com
spec:
  rules:
    - host: ose.test.s3.com
      http:
        paths:
          - backend:
              service:
                name: service-name
                port:
                  number: 8080
            path: /
            pathType: ImplementationSpecific
  tls:
    - hosts:
        - ose.test.s3.com
      secretName: secret-name
```

Now the question is, obviously I cannot visit the random host name I chose (`ose.test.s3.com`) in browser. 

If I use the command `kubectl port-forward service/service-name 8080:8080 -n ns`  to forward the service to localhost and visit it by `localhost:8080`, it's still under http protocol and it's not secure. Forcing it to be `https://localhost:8080` is not feasible.

## Solution

Get IP address of kind node’s Docker container:

``` bash
docker container inspect kind-control-plane \
  --format '{{ .NetworkSettings.Networks.kind.IPAddress }}'
```

Mine is *172.18.0.3*.

Create a Docker container, and leverage `docker run`’s `--add-host` argument to add an entry to the container’s `/etc/hosts` file. I can run curl command like this:

``` bash
docker run \
  --add-host ose.test.s3.com:172.18.0.3 \
  --net kind \
  --rm \
  curlimages/curl \
  curl -k -v https://ose.test.s3.com
```

Or openssl like this:

``` bash
docker run \
  --add-host ose.test.s3.com:172.18.0.3 \
  --net kind \
  --rm \
  nginx \
  openssl s_client -connect ose.test.s3.com:443
```

It is worth noting that openssl should connect to *ose.test.s3.com:443* rather than *https://ose.test.s3.com* to get the certificate.

Then I get the service's self-signed certificate and can view it by:

```bash
openssl x509 -in cert.pem -noout -text
```

Also, I can check the secret in cluster with:

``` bash
openssl x509 -in <(kubectl get secret secret-name -n ns \
    -o jsonpath='{.data.tls\.crt}' | base64 --decode) -noout -text
```

These two certificate should be the same.

## Reference

[How to Test Ingress in a kind cluster](https://dustinspecker.com/posts/test-ingress-in-kind/#create-a-kind-cluster-with-ingress-support)
