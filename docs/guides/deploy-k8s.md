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

Differences in deployment make concrete examples challenging, here it is explained only the guidelines.

For secure deployment of the system one can think it by what can be accessed from where, for all ways of deploying two trust boundaries can be used, external and internal. For an extra layer of security also the storage trust boundary can be separate. The service is provided for customers on the internet therefore an example of deploying the service is using two separate Kubernetes clusters, one for responding to customers and other communication from outside, and another, more secure, storage facing internal cluster.

One thing to consider is where to release the data, that could be closed protected environment with tightly restricted access. If Data out is used to serve unencrypted files the recommendation is to have it available only in an internal cluster.

The services could be divided into two trust boundaries
- The services in external cluster are [Inbox](/docs/submission.md#submission-inbox ) and [MQ](/docs/connection.md#local-message-broker)
- The services in internal cluster are [Intercept](/docs/services/intercept.md), [Ingest](/docs/services/ingest.md), [Verify](/docs/services/verify.md), [Mapper](/docs/services/mapper.md), [Finalize](/docs/services/finalize.md), [Backup](/docs/services/backup.md) and [Data out](/docs/dataout.md).

The innermost trust zone contains the database and the archive, which be can accessed only from internal cluster.



## Charts overview

### sda-db - Database component for Sensitive Data Archive (SDA) installation

This chart deploys a pre-configured database ([PostgreSQL](https://www.postgresql.org/)) instance for Sensitive Data Archive, the database schemas are designed to adhere to [European Genome-Phenome Archive](https://ega-archive.org/) federated archiving model.

### sda-mq - RabbitMQ component for Sensitive Data Archive (SDA) installation

This chart deploys a pre-configured message broker ([RabbitMQ](https://www.rabbitmq.com/)) designed to work [European Genome-Phenome Archive](https://ega-archive.org/) federated messaging interface between `CentralEGA` and Local/Federated EGAs.

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
