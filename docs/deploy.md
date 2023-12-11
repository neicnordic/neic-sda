Deployments and Local Bootstrap
===============================

We use different deployment strategies for environments like Docker
Swarm, Kubernetes or a local-machine. The [local development and testing](guides/local-dev-and-testing.md) guide is
recommended for local-machine, while
[Kubernetes](https://kubernetes.io/) and [Docker Swarm](https://docs.docker.com/engine/swarm/) are recommended for production.

The production deployment repositories are:

-   [Kubernetes Helm charts](https://github.com/neicnordic/sensitive-data-archive/tree/main/charts);
-   [Docker Swarm deployment](https://github.com/neicnordic/LocalEGA-deploy-swarm/).

`neicnordic/sensitive-data-archive`, provides the SDA services as well as PostgreSQL and RabbitMQ configuration. The following container image is used in the deployments where the tag separates between services:

- `ghcr.io/neicnordic/sensitive-data-archive:<version>-postgres` - PostgreSQL database
- `ghcr.io/neicnordic/sensitive-data-archive:<version>-rabbitmq` - RabbitMQ message broker
- `ghcr.io/neicnordic/sensitive-data-archive:<version>-sftp-inbox` - sftp inbox
- `ghcr.io/neicnordic/sensitive-data-archive:<version>-auth` - authentication service
- `ghcr.io/neicnordic/sensitive-data-archive:<version>-download` - download service
- `ghcr.io/neicnordic/sensitive-data-archive:<version>` - all other services such as: `finalize`, `ingest`, `intercept`, `verify`, `mapper`, `sync`, `syncapi` and `s3inbox`

Guides
------

Different nodes of the `FederatedEGA` network, and projects using the stand-alone SDA have made different decisions in how to deploy the system.
Adaptations needs to be made depending on the system to deploy on, as well as the requirements of your deployment.

- [Deploying with Docker Swarm](guides/deploy-swarm.md)
- [Deploying with Kubernetes](guides/deploy-k8s.md)
