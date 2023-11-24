# Deploying on Kubernetes

> TODO:
> This guide is a stub and has yet to be finished.
> If you have feedback to give on the content you would like to see, please contact us on
> [github](https://github.com/neicnordic/neic-sda)!


## Guide summary

This guide explains how to deploy the Sensitive Data Archive (SDA) in kubernetes.
- What it intends to cover
- What to expect, scope, explain level of details
- How self-contained the guide is
- Examples expected to work directly or not, must be configured (example configurations, most updated version?)

## Local security / zone considerations

- Differences in deployment make concrete examples challenges, explain what can be exemplified and what not in this guide

## Charts overview

### sda-db - Database component for Sensitive Data Archive (SDA) installation

This chart deploys a pre-configured database ([PostgreSQL](https://www.postgresql.org/)) instance for Sensitive Data Archive, the database schemas are designed to adhere to [European Genome-Phenome Archive](https://ega-archive.org/) federated archiving model.

### sda-mq - RabbitMQ component for Sensitive Data Archive (SDA) installation

This chart deploys a pre-configured message broker ([RabbitMQ](https://www.rabbitmq.com/)) designed to work [European Genome-Phenome Archive](https://ega-archive.org/) federated messaging interface between Central EGA and Local/Federated EGAs.

### sda-svc - Components for Sensitive Data Archive (SDA) installation

This chart deploys the service components needed to operate the Sensitive Data Archive solution for running a Federated EGA node.
The charts may include additional service components that might be beneficial for administrative operations or extending the Sensitive Data Archive solutions to facilitate other use cases.

## System requirements

 - kubernetes minimal version required for running the helm charts is `>= 1.25`
 - helm minimal version required for running the charts is `>=3.5`

### Resource estimation

- RabbitMQ - official recommended resource requirements for a [RabbitMQ cluster](https://www.rabbitmq.com/kubernetes/operator/using-operator.html#resource-reqs)
- PostgreSQL - official recommended resource requirements for [PostgreSQL](https://www.postgresql.org/docs/current/install-requirements.html)

#### Minimal working configuration

The table below reflects the minimum required resources to run the services in the helm charts.

| Service    | CPU   | Memory | Disk |
|------------|-------|--------|------|
| RabbitMQ   | 1000m | 1Gi    | 8Gi  |
| PostgreSQL | 100m  | 128Mi  | 8Gi  |
| intercept  | 100m  | 32Mi   | -    |
| ingest     | 100m  | 128Mi  | -    |
| verify     | 100m  | 128Mi  | -    |
| finalize   | 100m  | 128Mi  | -    |
| download   | 100m  | 256Mi  | -    |
| backup     | 100m  | 128Mi  | -    |
| auth       | 100m  | 128Mi  | -    |
| s3inbox    | 100m  | 128Mi  | -    |
| sftpinbox  | 100m  | 128Mi  | -    |
| doa        | 100m  | 128Mi  | -    |

## Security issues

 - Enabling TLS example
 - Secret handling example

## Network policies

 - DNS names and ingress for services

## Complementary services

 - sda-auth, sda-doa, sda-download
