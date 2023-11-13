# NeIC Sensitive Data Archive

Recommended provisioning methods provided for production are:

* on a [Kubernetes cluster](https://github.com/neicnordic/sensitive-data-archive/tree/main/charts), using `kubernetes` and `helm` charts;
* on a [Docker Swarm cluster](https://github.com/neicnordic/LocalEGA-deploy-swarm), using `gradle` and `docker swarm`.

## Architecture

SDA is divided into several components, which can be deployed either for Federated EGA or as an stand-alone SDA.

### Core Components

Source code for core components is available at: https://github.com/neicnordic/sensitive-data-archive

| Component     | Role |
|---------------|------|
| inbox         | SFTP, S3 or HTTPS server, acting as a dropbox, where user credentials are fetched from CentralEGA or via LifeScience AAI. [s3inbox](https://github.com/neicnordic/sensitive-data-archive/tree/main/sda/cmd/s3inbox/s3inbox.md) or [sftp-inbox](https://github.com/neicnordic/sensitive-data-archive/tree/main/sda-sftp-inbox) |
| intercept     | The intercept service relays message between the queue provided from the federated service and local queues. **(Required for Federated EGA use case)** |
| ingest        | Split the Crypt4GH header and move the remainder to the storage backend. No cryptographic task, nor access to the decryption keys. |
| verify        | Decrypt the stored files and checksum them against their embedded checksum. |
| archive       | Storage backend: as a regular file system or as a S3 object store. |
| finalize      | Handle the so-called _Accession ID_ to filename mappings from CentralEGA. |
| mapper        | The mapper service register mapping of accessionIDs (IDs for files) to datasetIDs. |
| data out API  | Provides a download/data access API for streaming archived data either in encrypted or decrypted format - source at: https://github.com/neicnordic/sda-doa |
| download      | Provides a download/data access API for streaming (decrypted) archived data - source at: https://github.com/neicnordic/sda-download |

### Associated components

| Component     | Role |
|---------------|------|
| db            | A [Postgres database](https://github.com/neicnordic/sensitive-data-archive/tree/main/postgresql) with appropriate schemas and isolations |
| mq            | A [(local) RabbitMQ](https://github.com/neicnordic/sensitive-data-archive/tree/main/rabbitmq) message broker with appropriate accounts, exchanges, queues and bindings, connected to the CentralEGA counter-part. |


### Stand-alone components

| Component     | Role |
|---------------|------|
| orchestrate   | Component that automates ingestion in stand-alone deployments of SDA Pipeline https://github.com/neicnordic/sda-orchestration |
