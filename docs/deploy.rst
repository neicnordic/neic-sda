Deployments and Local Bootstrap
===============================

We use different deployment strategies for environments
like Docker Swarm, Kubernetes or a local-machine. The local machine 
environment is recommended for development and testing, while `Kubernetes <https://kubernetes.io/>`_
and `Docker Swarm <https://docs.docker.com/engine/swarm/>`_ for production. 

The production deployment repositories are:

* `Kubernetes Helm charts <https://github.com/neicnordic/sda-helm/>`_;
* `Docker Swarm deployment <https://github.com/neicnordic/LocalEGA-deploy-swarm/>`_.

The following container images are used in the deployments:

* ``neicnordic/sda-base``, provides the LocalEGA services (based on `python:3.6-alpine3.10`);
* ``neicnordic/sda-mq``, provides the broker (mq) service (based on `rabbitmq:3.7.8-management`);
* ``neicnordic/sda-db``, provides the database service (based on `postgres:11.2`);
* ``neicnordic/sda-inbox-sftp``, provides the inbox service via sftp (based on Apache Mina);
* ``neicnordic/sda-doa``, provides the data out service (Data Out API);
* ``neicnordic/sda-s3-proxy``, provides the inbox service via a s3 proxy (S3 proxy inbox).

