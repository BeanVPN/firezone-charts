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
