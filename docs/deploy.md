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

The following container images are used in the deployments:

-   `neicnordic/sensitive-data-archive`, provides the SDA services as well as PostgreSQL and RabbitMQ configuration.
