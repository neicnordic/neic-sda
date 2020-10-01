# NeIC Sensitive Data Archive

The [code](lega) is written in Python (3.6+).

Recommended provisioning methods provided for production are:

* on a [Kubernetes cluster](https://github.com/neicnordic/sda-helm/), using `kubernetes` and `helm` charts;
* on a [Docker Swarm cluster](https://github.com/neicnordic/LocalEGA-deploy-swarm), using `gradle` and `docker swarm`.

# Architecture

SDA is divided into several components, as docker containers.

| Component     | Role |
|---------------|------|
| db            | A Postgres database with appropriate schemas and isolations |
| mq            | A (local) RabbitMQ message broker with appropriate accounts, exchanges, queues and bindings, connected to the CentralEGA counter-part. |
| inbox         | SFTP, S3 or HTTPS server, acting as a dropbox, where user credentials are fetched from CentralEGA or via ELIXIR AAI. |
| ingest        | Split the Crypt4GH header and move the remainder to the storage backend. No cryptographic task, nor access to the decryption keys. |
| verify        | Decrypt the stored files and checksum them against their embedded checksum. |
| archive       | Storage backend: as a regular file system or as a S3 object store. |
| finalize      | Handle the so-called _Accession ID_ to filename mappings from CentralEGA. |
| data out API  | Provides a download/data access API for streaming archived data either in encrypted or decrypted format - source at: https://github.com/neicnordic/LocalEGA-DOA |
| metadata      | Component used in standalone version of SDA. Provides an interface and backend to submit Metadata and associated with a file in the Archive. _source not part of this repo_ |

Find the [NeIC SDA documentation](https://neic-sda.readthedocs.io) hosted on [ReadTheDocs.org](https://readthedocs.org/).
