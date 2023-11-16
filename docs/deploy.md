Deployments and Local Bootstrap
===============================

We use different deployment strategies for environments like Docker
Swarm, Kubernetes or a local-machine. The local machine environment is
recommended for development and testing, while
[Kubernetes](https://kubernetes.io/) and [Docker
Swarm](https://docs.docker.com/engine/swarm/) for production.

The production deployment repositories are:

-   [Kubernetes Helm charts](https://github.com/neicnordic/sensitive-data-archive/tree/main/charts);
-   [Docker Swarm
    deployment](https://github.com/neicnordic/LocalEGA-deploy-swarm/).

`neicnordic/sensitive-data-archive`, provides the SDA services as well as PostgreSQL and RabbitMQ configuration. The following container image is used in the deployments where the tag separates between services:

- `ghcr.io/neicnordic/sensitive-data-archive:<version>-postgres` - PostgreSQL database
- `ghcr.io/neicnordic/sensitive-data-archive:<version>-rabbitmq` - RabbitMQ message broker
- `ghcr.io/neicnordic/sensitive-data-archive:<version>-sftp-inbox` - sftp inbox
- `ghcr.io/neicnordic/sensitive-data-archive:<version>-auth` - authentication service
- `ghcr.io/neicnordic/sensitive-data-archive:<version>-download` - download service
- `ghcr.io/neicnordic/sensitive-data-archive:<version>` - all other services such as: `finalize`, `ingest`, `intercept`, `verify`, `mapper` and `s3inbox`
