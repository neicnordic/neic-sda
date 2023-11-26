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
| auth       | 100m  | 128Mi  | -    |
| s3inbox    | 100m  | 128Mi  | -    |
| sftpinbox  | 100m  | 128Mi  | -    |
| doa        | 100m  | 128Mi  | -    |

## Minimal set of configuration variables

Here we provide a minimal list of variables that need to be configured in the [values.yml](https://github.com/neicnordic/sensitive-data-archive/blob/main/charts/sda-svc/values.yaml) file of the Helm charts in order to achieve a working deployment of the `sensitive data archive` [services](https://github.com/neicnordic/sensitive-data-archive/tree/main/charts/sda-svc). In the following it is assumed that a *federated* setup is being deployed.

### Global Variables

TLS support:
- `global.tls.issuer` or `glbal.tls.clusterIssuer`

Storage:
- `global.archive.storageType`: The storage type for the archive. If this is set to `s3`, then the following variables need to be set as well:
- `global.archive.s3Url`: The S3 URL for the archive.
- `global.archive.s3Bucket`: The S3 bucket for the archive.
- `global.archive.s3AccessKey`: The S3 access key for the archive.
- `global.archive.s3SecretKey`: The S3 secret key for the archive.

- `global.backupArchive.storageType`: The storage type for the backup archive. If this is set to `s3`, then the following variables need to be set as well:
- `global.backupArchive.s3Url`: The S3 URL for the backup archive.
- `global.backupArchive.s3Bucket`: The S3 bucket for the backup archive.
- `global.backupArchive.s3AccessKey`: The S3 access key for the backup archive.
- `global.backupArchive.s3SecretKey`: The S3 secret key for the backup archive.

- `global.inbox.storageType`: The storage type for the inbox. If this is set to `s3`, then the following variables need to be set as well:
- `global.inbox.s3Url`: The S3 URL for the inbox.
- `global.inbox.s3Bucket`: The S3 bucket for the inbox.
- `global.inbox.s3AccessKey`: The S3 access key for the inbox.
- `global.inbox.s3SecretKey`: The S3 secret key for the inbox.

RabbitMQ:
- `global.broker.host`: The host for the broker.
- `global.broker.exchange`: The exchange for the broker.
- `global.broker.routingError`: The routing error for the broker.
- `global.broker.backupRoutingKey`: The backup routing key for the broker.

Crypt4gh:
- `global.c4gh.secretName`: The name by which the kubernetes secret for c4gh is referenced in the Helm charts.
- `global.c4gh.keyFile`: The crypt4gh private key file.
- `global.c4gh.passphrase`: The passphrase for c4gh.

CEGA:
- `global.cega.host`: The host for [Federated EGA NSS API](https://nss.ega-archive.org/spec/#/).
- `global.cega.user`: The user for accessing Federated EGA NSS API.
- `global.cega.password`: The password for Federated EGA NSS API.

Database:
- `global.db.host`: The host for the database.

## Service Specific Credentials

Intercept:
- `credentials.intercept.mqUser`: The message queue user for intercept.
- `credentials.intercept.mqPassword`: The message queue password for intercept.

Ingest:
- `credentials.ingest.dbUser`: The database user for ingest.
- `credentials.ingest.dbPassword`: The database password for ingest.
- `credentials.ingest.mqUser`: The message queue user for ingest.
- `credentials.ingest.mqPassword`: The message queue password for ingest.

Verify:
- `credentials.verify.dbUser`: The database user for verify.
- `credentials.verify.dbPassword`: The database password for verify.
- `credentials.verify.mqUser`: The message queue user for verify.
- `credentials.verify.mqPassword`: The message queue password for verify.

Finalize:
- `credentials.finalize.dbUser`: The database user for finalize.
- `credentials.finalize.dbPassword`: The database password for finalize.
- `credentials.finalize.mqUser`: The message queue user for finalize.
- `credentials.finalize.mqPassword`: The message queue password for finalize.

Mapper:
- `credentials.mapper.dbUser`: The database user for mapper.
- `credentials.mapper.dbPassword`: The database password for mapper.
- `credentials.mapper.mqUser`: The message queue user for mapper.
- `credentials.mapper.mqPassword`: The message queue password for mapper.

If Backup functionality is enabled:
- `credentials.backup.dbUser`: The database user for backup.
- `credentials.backup.dbPassword`: The database password for backup.
- `credentials.backup.mqUser`: The message queue user for backup.
- `credentials.backup.mqPassword`: The message queue password for backup.

## Minimal configuration for additional services

Ingress:
- `global.ingress.deploy`: Determines if the ingress should be deployed.
- `global.ingress.clusterIssuer`: The cluster issuer for the ingress.
- `global.ingress.hostName.auth`: The hostname for the auth.
- `global.ingress.hostName.download`: The hostname for the download.
- `global.ingress.hostName.s3Inbox`: The hostname for the S3 Inbox.

SDA-auth:
- `global.auth.jwtSecret`: The JWT secret for auth.
- `global.auth.jwtKey`: The JWT key for auth.
- `global.auth.jwtPub`: The JWT public key for auth.

SDA-download:
- `global.download.enabled`: Determines if the download is enabled.
- `credentials.download.dbUser`: The database user for download.
- `credentials.download.dbPassword`: The database password for download.

LS-AAI OIDC:
- `global.oidc.id`: The ID for OIDC.
- `global.oidc.secret`: The secret for OIDC.

S3Inbox:
- `credentials.inbox.dbUser`: The database user for inbox.
- `credentials.inbox.dbPassword`: The database password for inbox.
- `credentials.inbox.mqUser`: The message queue user for inbox.
- `credentials.inbox.mqPassword`: The message queue password for inbox.

## Security issues

 - Enabling TLS example
 - Secret handling example

## Network policies

 - DNS names and ingress for services

## Complementary services

 - sda-auth, sda-doa, sda-download
