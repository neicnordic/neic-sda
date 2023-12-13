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

One thing to consider is where to release the data, that could be closed protected environment with tightly restricted access. If Data Retrieval API is used to serve unencrypted files the recommendation is to have it available only in an internal cluster.

The services could be divided into two trust boundaries
- The services in external cluster are [Inbox](/docs/submission.md#submission-inbox ) and [MQ](/docs/connection.md#local-message-broker)
- The services in internal cluster are [Intercept](/docs/services/intercept.md), [Ingest](/docs/services/ingest.md), [Verify](/docs/services/verify.md), [Mapper](/docs/services/mapper.md), [Finalize](/docs/services/finalize.md), [Backup](/docs/services/backup.md) and [Data Retrieval API](/docs/dataout.md).

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

Here we provide minimal lists of variables that need to be configured, in addition to the defaults, in the respective `values.yml` file of each of the Helm charts for:

- [SDA services](#sda-services-chart)
- [RabbitMQ](#rabbitmq-chart)
- [SDA Database](#database-chart)

in order to achieve a working deployment of the `sensitive data archive`. In the following it is assumed that a *federated* setup is being deployed.

### SDA services chart

Below is a minimal list of variables that need to be configured in the [values.yml](https://github.com/neicnordic/sensitive-data-archive/blob/main/charts/sda-svc/values.yaml) file of the Helm chart for the `sensitive data archive` services in order to achieve a working deployment. Detailed documentation on all of the chart's variables can be found [here](https://github.com/neicnordic/sensitive-data-archive/tree/main/charts/sda-svc).

#### Global Variables

##### TLS support

- `global.tls.issuer` or `global.tls.clusterIssuer`: The issuer or cluster issuer for TLS

##### Storage

- `global.archive.storageType`: The storage type for the archive
- `global.backupArchive.storageType`: The storage type for the backup archive.
- `global.inbox.storageType`: The storage type for the inbox.

  If, for example the above are set to `s3`, then the following variables need to be set as well:

- `global.archive.s3Url`: The S3 URL for the archive
- `global.archive.s3Bucket`: The S3 bucket for the archive
- `global.archive.s3AccessKey`: The S3 access key for the archive
- `global.archive.s3SecretKey`: The S3 secret key for the archive

- `global.backupArchive.s3Url`: The S3 URL for the backup archive
- `global.backupArchive.s3Bucket`: The S3 bucket for the backup archive
- `global.backupArchive.s3AccessKey`: The S3 access key for the backup archive
- `global.backupArchive.s3SecretKey`: The S3 secret key for the backup archive

- `global.inbox.s3Url`: The S3 URL for the inbox
- `global.inbox.s3Bucket`: The S3 bucket for the inbox
- `global.inbox.s3AccessKey`: The S3 access key for the inbox
- `global.inbox.s3SecretKey`: The S3 secret key for the inbox

##### RabbitMQ

- `global.broker.host`: The host for the broker
- `global.broker.exchange`: The exchange for the broker
- `global.broker.routingError`: The routing error for the broker
- `global.broker.backupRoutingKey`: The backup routing key for the broker

##### Crypt4gh

- `global.c4gh.secretName`: The name by which the kubernetes secret for c4gh is referenced in the Helm charts
- `global.c4gh.keyFile`: The crypt4gh private key file
- `global.c4gh.passphrase`: The passphrase for c4gh

##### CEGA

- `global.cega.host`: The host for [Federated EGA NSS API](https://nss.ega-archive.org/spec/#/)
- `global.cega.user`: The user for accessing Federated EGA NSS API
- `global.cega.password`: The password for Federated EGA NSS API

##### Database

- `global.db.host`: The host for the database

#### Service Specific Credentials

##### Intercept

- `credentials.intercept.mqUser`: The message queue user for intercept
- `credentials.intercept.mqPassword`: The message queue password for intercept

##### Ingest

- `credentials.ingest.dbUser`: The database user for ingest
- `credentials.ingest.dbPassword`: The database password for ingest
- `credentials.ingest.mqUser`: The message queue user for ingest
- `credentials.ingest.mqPassword`: The message queue password for ingest

##### Verify

- `credentials.verify.dbUser`: The database user for verify
- `credentials.verify.dbPassword`: The database password for verify
- `credentials.verify.mqUser`: The message queue user for verify
- `credentials.verify.mqPassword`: The message queue password for verify

##### Finalize

- `credentials.finalize.dbUser`: The database user for finalize
- `credentials.finalize.dbPassword`: The database password for finalize
- `credentials.finalize.mqUser`: The message queue user for finalize
- `credentials.finalize.mqPassword`: The message queue password for finalize

To enable Backup functionality:

- `credentials.backup.dbUser`: The database user for backup
- `credentials.backup.dbPassword`: The database password for backup
- `credentials.backup.mqUser`: The message queue user for backup
- `credentials.backup.mqPassword`: The message queue password for backup

##### Mapper

- `credentials.mapper.dbUser`: The database user for mapper
- `credentials.mapper.dbPassword`: The database password for mapper
- `credentials.mapper.mqUser`: The message queue user for mapper
- `credentials.mapper.mqPassword`: The message queue password for mapper

#### Minimal configuration for additional services

##### Ingress

- `global.ingress.deploy`: Determines if the ingress should be deployed
- `global.ingress.clusterIssuer`: The cluster issuer for the ingress
- `global.ingress.hostName.auth`: The hostname for the auth
- `global.ingress.hostName.download`: The hostname for the download
- `global.ingress.hostName.s3Inbox`: The hostname for the S3 Inbox

##### SDA-auth

- `global.auth.jwtSecret`: The JWT secret for auth
- `global.auth.jwtKey`: The JWT key for auth
- `global.auth.jwtPub`: The JWT public key for auth

##### SDA-download

- `global.download.enabled`: Determines if the download is enabled
- `credentials.download.dbUser`: The database user for download
- `credentials.download.dbPassword`: The database password for download

##### LS-AAI OIDC

- `global.oidc.id`: The ID for OIDC
- `global.oidc.secret`: The secret for OIDC

##### S3Inbox

- `credentials.inbox.dbUser`: The database user for inbox
- `credentials.inbox.dbPassword`: The database password for inbox
- `credentials.inbox.mqUser`: The message queue user for inbox
- `credentials.inbox.mqPassword`: The message queue password for inbox

### RabbitMQ chart

Below is a minimal list of variables that need to be configured in the [values.yml](https://github.com/neicnordic/sensitive-data-archive/blob/main/charts/sda-mq/values.yaml) file of the `RabbitMQ` Helm chart in order to achieve a working deployment. Detailed documentation on all of the chart's variables can be found [here](https://github.com/neicnordic/sensitive-data-archive/tree/main/charts/sda-mq).

- `global.adminUser`: The username for the admin user
- `global.adminPassword`: The password for the admin user
- `global.shovel.host`: The hostname of the shovel server
- `global.shovel.pass`: The password to authenticate with the shovel server
- `global.shovel.port`: The port on which the shovel server is running
- `global.shovel.user`: The username to authenticate with the shovel server
- `global.shovel.vhost`: The virtual host on the shovel server

### Database chart

Below is a minimal list of variables that need to be configured in the [values.yml](https://github.com/neicnordic/sensitive-data-archive/blob/main/charts/sda-db/values.yaml) file of the `SDA Database` Helm chart in order to achieve a working deployment. Detailed documentation on all of the chart's variables can be found [here](https://github.com/neicnordic/sensitive-data-archive/tree/main/charts/sda-db).

- `global.postgresAdminPassword`: The password for the postgres admin user
- `global.tls.clusterIssuer`: The cluster issuer for TLS
- `global.tls.secretName`: The name by which the kubernetes secret for TLS is referenced in the Helm charts

## Security issues

 - Enabling TLS example
 - Secret handling example

## Network policies

 - DNS names and ingress for services
 
    When deploying applications on Kubernetes, it is essential to understand the DNS naming conventions and ingress configurations for [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [Services](https://kubernetes.io/docs/concepts/services-networking/service/). Each Pod within the cluster is assigned a [DNS name](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) in the format of `pod-ip-address.<cluster>.pod.cluster.local`. This DNS resolution allows seamless communication between Pods within the same cluster.

   Services, representing sets of Pods, are assigned A DNS records with names structured as `<service_name>.<namespace>.svc.cluster.local`. This DNS record resolves to the cluster IP of the respective Service.

    | Service Name | Common DNS Name                         |
    | ------------ | ----------------------------------------|
    | inbox        | sda-svc-inbox.<namespace>.svc.cluster.local   |
    | download     | sda-svc-download.<namespace>.svc.cluster.local|
    | auth         | sda-svc-auth.<namespace>.svc.cluster.local    |
    | mq           | broker-sda-mq.<namespace>.svc.cluster.local   |

    Certain services, such as `inbox`, `download`, and `auth`, are configured to expect an ingress. [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) provides external access to these services, allowing external clients to communicate with them. The following services specifically expect an ingress:

    - inbox
    - download
    - auth

    In addition, Kubernetes allows you to define [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) to control the communication between Pods. Network Policies are crucial for enforcing security measures within your cluster. They enable you to specify which Pods can communicate with each other and define rules for ingress and egress traffic.
    Here are two recommended basic examples of a Network Policy for namespace isolation and allowing traffic to inbox ingress, a similar policies needs to be in place for `download` and `auth` service:

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: namespace-isolation
    spec:
      podSelector: {}
      ingress:
      - from:
        - podSelector:
            matchLabels:
              app.kubernetes.io/component: controller
          namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
      - from:
        - podSelector:
            matchLabels:
              app.kubernetes.io/component: controller
          namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx-direct
      policyTypes:
      - Ingress
    ```

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
        name: allow-inbox-ingress-outside-cluster
    spec:
        podSelector: 
            matchLabels:
            app: sda-svc-inbox
        ingress:
        - from:
            - ipBlock:
                cidr: 0.0.0.0/0
        policyTypes:
        - Ingress
    ```

## Complementary services

 - sda-auth, sda-doa, sda-download
