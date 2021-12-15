# postal

A helm-chart to deploy [Postal](https://github.com/postalserver/postal) on kubernetes.

## Introduction

This chart bootstraps a deployment of Postal, MariaDB and RabbitMQ on a
[Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.9
- PV provisioner support in the underlying infrastructure

## Optional prerequisites

- An ingress controller
- A functioning [cert-manager](https://github.com/jetstack/cert-manager) for certificate management

## Installing the Chart

To install the chart with the release name `my-release`, add the helm charts repository:

```console
clone this repository
```

Setup an ingress-controller

```console
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

```console
helm install my-ingress ingress-nginx/ingress-nginx `
     --namespace ingress `
     --create-namespace `
     --set controller.replicaCount=2 `
     --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux `
     --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

Update your domain info using the IP from ingress controller to add to your @/* (A) dns record

Install cert-manager

```console
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.6.1/cert-manager.yaml
```

Install our included issuer or create your own https://cert-manager.io/docs/concepts/issuer/

```console
kubectl apply -f issuer-dev.yaml 
```

When changing from dev to prod
- Uninstall postal helm chart
- Delete clusterissuer and it's secret from cert-manager namespace
- Delete postal-smtp-tls and postal-web-tls from default namespace
- Install dev or prod issuer then install as normal

Override ingress-controller values to add smtp port
 
```console
helm upgrade --install -n ingress ingress-controller ingress-nginx/ingress-nginx --values nginxvalues.yaml --wait
```

- Change values in values.yaml to match your domain/dns/installation settings

- Change default passwords

- and install the chart

```console
helm install my-release .
```

The command deploys postal on the Kubernetes cluster in the default configuration. The [configuration](#configuration)
section lists parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration - MariaDB and RabbitMQ

This chart pulls in [mariadb](https://github.com/bitnami/charts/tree/master/bitnami/mariadb) and
[rabbitmq]https://github.com/bitnami/charts/tree/master/bitnami/rabbitmq) as dependencies.

Please refer to the respective documentation for configuration parameters of those components.

## Configuration

We set the following default configuration parameters for MariaDB and RabbitMQ:

Parameter | Description | Default
--- | --- | ---
`mariadb.auth.rootPassword` | Password for the MariaDB root user. Change this from the default! | see [values.yaml](values.yaml)
`mariadb.replication.enabled` | Enable MariaDB replication | `false`
`mariadb.slave.replicas` | Number of MariaDB slave replicas | `0`
`mariadb.metrics.enabled` | Enable prometheus metrics | `true`
`rabbitmq.extraConfiguration.default_vhost` | RabbitMQ vhosts definitions. Our default adds one for Postal. | see [values.yaml](values.yaml)
`rabbitmq.default_permissions` | RabbitMQ vhosts permissions. Our default adds permission to the `/postal` vhost for the `postal` user. | see [values.yaml](values.yaml)
`rabbitmq.replicaCount` | Number of RabbitMQ replicas. | `1`
`rabbitmq.auth.username` | Username for RabbitMQ. | `postal`
`rabbitmq.auth.password` | Password for RabbitMQ. Change this from the default! | see [values.yaml](values.yaml)
`rabbitmq.auth.password` | Password for RabbitMQ management operations. Change this from the default! | see [values.yaml](values.yaml)

The following table lists the configurable parameters of the postal chart and their default values.

Parameter | Description | Default
--- | --- | ---
`postal.nameOverride` | override the name of the chart | ``
`postal.config` | A postal configuration yaml to apply on top of postal's default configuration. See [Postal's default configuration](https://github.com/atech/postal/blob/master/config/postal.defaults.yml) for available options. | `{}`
`postal.image` | postal container image repository | `linkyard/postal`
`postal.imageTag` | postal container image tag | `latest`
`postal.imagePullPolicy` | postal container image pull policy | `Always`
`postal.resources` | CPU/Memory resource requests/limits  | `{}`
`postal.signingKey` | RSA private key in PEM format used for DKIM signing. Change this from the default! | see [values.yaml](values.yaml)
`postal.railsSecretKey` | The secret key for rails. Change this from the default! | see [values.yaml](values.yaml)
`postal.letsEncryptKey` | RSA private key in PEM format. Used by Postal to acquire and renew certificates for the click-tracking-server from Let's Encrypt. Change this from the default! | see [values.yaml](values.yaml)
`postal.smtpPassword` | Password for the SMTP server. Change this from the default! | see [values.yaml](values.yaml)
`postal.web.ingress.enabled` | if an `ingress` resource should be deployed for the web interface | `true`
`postal.web.ingress.hostname` | public hostname for the web interface; this is a required value  | ``
`postal.web.ingress.ingressClass` | ingress class to use | `nginx`
`postal.web.ingress.tlsEnabled` | enable TLS on the ingress | `true`
`postal.web.ingress.certManager.enabled` | enable management of the TLS secret with [cert-manager](https://github.com/jetstack/cert-manager) | `true`
`postal.web.ingress.certManager.ingressClass` | ingress class to use for HTTP01 challenge | `nginx`
`postal.web.ingress.certManager.issuerName` | name of the cert-manager issuer; this is a required value | ``
`postal.web.ingress.certManager.issuerKind` | kind of the cert-manager issuer; this is a required value | ``
`postal.web.ingress.existingTlsSecret` | name of an existing TLS secret to use for the ingress (if cert-manager is not used); must be in the same namespace | ``
`postal.smtp.hostname` | public hostname of postal's SMTP server; this is a required value | ``
`postal.smtp.serviceType` | what kind of service the SMTP server is exposed as | `ClusterIP`
`postal.smtp.certManager.enabled` | enable management of the TLS secret with [cert-manager](https://github.com/jetstack/cert-manager) | `true`
`postal.smtp.certManager.ingressClass` | ingress class to use for HTTP01 challenge | `nginx`
`postal.smtp.certManager.issuerName` | name of the cert-manager issuer; this is a required value | ``
`postal.smtp.certManager.issuerKind` | kind of the cert-manager issuer; this is a required value | ``
`postal.smtp.existingTlsSecret` | name of an existing TLS secret to use for the SMTP server (if cert-manager is not used); must be in the same namespace | ``