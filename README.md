# Firezone chart
[Firezone](https://www.firezone.dev/) is an open-source remote access platform built on WireGuard®, a modern VPN protocol that's 4-6x faster than OpenVPN. Deploy on your infrastructure and start onboarding users in minutes.

## TL;DR

```console
helm install firezone ./firezone
```

## Introduction

This chart bootstraps a [Firezone](https://www.firezone.dev/) deployment on a Kubernetes cluster using the Helm package manager.

## Prerequisites

- Kubernetes 1.21+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart with the release name `my-release`:

```console
helm install my-release ./firezone
```

The command deploys PostgreSQL on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
helm delete my-release
```

The command removes all the Kubernetes components but PVC's associated with the chart and deletes the release.

To delete the PVC's associated with `my-release`:

```console
kubectl delete pvc -l release=my-release
```

> **Note**: Deleting the PVC's will delete postgresql data as well. Please be cautious before doing it.

## Parameters
### Firezone parameters

| Name                     | Description                                                                                      | Value              |
| ------------------------ | ------------------------------------------------------------------------------------------------ | ------------------ |
| `adminEmail `            | Primary administrator email.                                                                     | `""`               |
| `externalUrl`            | The external URL the web UI will be accessible at                                                | `""`               |
| `endpoint`               | IPv4, IPv6 address, or FQDN that devices will be configured to connect to                        | `""`               |
| `adminPassword`          | Default password that will be used for creating or resetting the primary administrator account.  | `""`               |
| `image.repository`       | Firezone image repository                                                                        | `firezone/firezone`|
| `image.tag`              | Firezone image tag                                                                               | `""`               |
| `image.pullPolicy`       | Firezone image pull policy                                                                       | `IfNotPresent`     |


### PostgreSQL common parameters

| Name                           | Description                                        | Value           |
| ------------------------------ | -------------------------------------------------- | --------------- |
| `postgresql.auth.database`     | Name for a Firezone database to create             | `firezone`    |
| `postgresql.auth.username`     | Name for a Firezone user to create                 | `firezone`    |
| `postgresql.auth.password`     | Password for the Firezone user to create           | `""`            |


# Setup MicroK8s to run Firezone
## Configure pod CIDR for Calico CNI https://microk8s.io/docs/configure-cni#configure-pod-cidr-6
The default CIDR for pods is 10.1.0.0/16

The default service CIDR is 10.152.183.0/24. 10.152.183.1 will typically be reserved for the Kubernetes API, and 10.152.183.10 will be used by CoreDNS.

If the default CIDR of pod overlaps with the host network, we need to change it
- Edit /var/snap/microk8s/current/args/cni-network/cni.yaml
```
# in daemonset/calico-node/containers[calico-node]/env
            - name: CALICO_IPV4POOL_CIDR
              value: "10.1.0.0/16"                     # Change "10.1.0.0/16" to "10.100.0.0/16"
```
- Also, edit /var/snap/microk8s/current/args/kube-proxy and set the --cluster-cidr accordingly:
```
--cluster-cidr=10.1.0.0/16                            # Change "10.1.0.0/16" to "10.100.0.0/16"
```
- Then, restart MicroK8s and re-apply Calico with:
```
$ microk8s kubectl apply -f /var/snap/microk8s/current/args/cni-network/cni.yaml
$ sudo snap restart microk8s
```
- If Calico has already started and created a default IPPool, you might have to delete it with:
```
$ microk8s kubectl delete ippools default-ipv4-pool
$ microk8s kubectl rollout restart daemonset/calico-node
```

## Configure dns https://microk8s.io/docs/addon-dns
At default, coredns addon of microk8s pod will use the resolv.conf at /run/systemd/resolve/resolv.conf
If you want to use another dns servers, you can run these commands

```
$ microk8s disable dns
$ microk8s enable dns:8.8.8.8,8.8.4.4
```

## Install nginx ingress controller and cert-manager https://blog.antosubash.com/posts/setup-nginx-and-cert-manager-in-micro-k8s

```
$ microk8s enable cert-manager

$ microk8s kubectl apply -f - <<EOF
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: cert@peterbean.net
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-prod
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: public
EOF


$ microk8s enable ingress

$ microk8s kubectl apply -f - <<EOF
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: http-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt"
    kubernetes.io/ingress.class: "public"
spec:
  tls:
    - hosts:
      - kimtruong.firezone.sandbox.peterbean.net
      secretName: ingress-tls
  rules:
  - host: "kimtruong.firezone.sandbox.peterbean.net"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: firezone
            port:
              number: 13000
EOF
```