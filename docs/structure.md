Structure
=========

This section provides overview material for how the services can be deployed in different constellations to build the archive functionality, and highlights communication paths between components.


Data Flow Overview
------------------

The overall data workflow consists of three parts:

-   The users logs onto the `FederatedEGA`'s inbox and uploads the encrypted files. They then go to the `CentralEGA`'s interface to prepare a submission;
-   Upon submission completion, the files are ingested into the archive and become searchable by the `CentralEGA`'s engine;
-   Once the file has been successfully archived, it can be accessed by researchers in accordance with permissions given by the corresponding Data Access Committee.

------------------------------------------------------------------------

`CentralEGA` contains a database of users with permissions to upload to a specific Sensitive Data Archive. The `CentralEGA` ID is used to
authenticate the user against either their EGA password or a private key.

For every uploaded file, `CentralEGA` receives a notification that the file is present in a SDA's inbox. The uploaded file must be encrypted
in the [Crypt4GH file format](https://samtools.github.io/hts-specs/crypt4gh.pdf) using that SDA public Crypt4gh key. The file is
checksumed and presented in the `CentralEGA`'s interface in order for the user to double-check that it was properly uploaded.

More details about process in [Data Submission](submission.md#data-submission).

When a submission is ready, `CentralEGA` triggers an ingestion process on the user-chosen SDA instance. `CentralEGA`'s interface is updated with
progress notifications whether the ingestion was successful, or whether there was an error.

More details about the [Ingestion Workflow](submission.md#ingestion-workflow).

Once a file has been successfully submitted and the ingestion process has been finalised, including receiving an `Accession ID` from Central
EGA. The Data Retrieval API can be utilised to retrieve set file by utilising the `Accession ID`. More details in [Data Retrieval API](dataout.md#data-retrieval-api).


Inter-communication between services
------------------------------------

There are three main ways that the system is passing on information and persist state in the system:

1. through AMQP messages sent from and to micro services;
2. changes in the database of the status of a file being processed via the the `sda-pipeline`;
3. location and state of files in either of the three file storage areas.

### AMQP messaging - Rabbit MQ

The orchestration of any action to be performed by the micro services, is managed through the appropriate AMQP message being posted at the RabbitMQ broker service, from where microservices will pick up messages about work to be performed. Each microservice will thus normally only process one type of messages/jobs from a specific AMQP queue, and have a predefined type of next message to post once the current task is completed, for the next microservice in the pipeline to carry on the next action needed.

### Database

The state of files being ingested to the SDA is recorded in a PostgreSQL database, and the different microservices will often update records in the database as part of their processing step in the pipeline.

### Inbox - Archive - Outbox areas

The SDA operates with three file areas (excluding additional backup mechanisms for data redundancy). The Inbox area is where users will be allowed to upload their encrypted files temporarily, before they get further processed into the archive. The uploaded files are then securely transferred into the Archive area with the header split off and stored in the database, after a validation of content integrity. If someone is later granted access to retrieve a file from the archive, the header is re-encrypted for the requester and merged back with the main content and stored in the Outbox area for the requester to retrieve it from there.


Additional components
---------------------

### Authentication of users

In a Federated setup, a data submitter will usually be required to have a user profile with the `CentralEGA` services as well as a user identity trusted by the Federated EGA node services.The [Life Science AAI](https://lifescience-ri.eu/) login identity is primarily used (a.k.a. ELIXIR AAI identity) for the latter. Integration towards both authentication services will likely need to be incorporated into a Federated EGA nodes upload mechanism and download mechanism.

### Authorizing access to datasets

SDA has two main implementations for serving datasets to requesters, both requiring a GA4GH Passport with a signed VISA from a trusted party to release a given dataset to the holder of the Passport.  


### Auxiliary, mock, utility services

In addition to the core microservices of the SDA solution, there are many auxiliary and utility services useful for testing and alternative deployments, but not required for a fully functional system.


Deployment related choices
--------------------------

### Federated vs stand-alone

In a Federated setup, the `FederatedEGA` archive node setup locally need to exchange status updates with the `CentralEGA` in a synchronized manner to basically orchestrate two parallel processes:

1. The multi-step process of uploading and safely archiving encrypted files holding both sensitive phenome and genome data.

2. The process of the Submitter annotating the archived data in an online portal at `CentralEGA`, resulting in assigned accession numbers for items such as DataSet, Study, Files etc.


In a stand-alone setup, the deployed service has less remote synchronisation to worry about (there is no dependency on `CentralEGA` provided services), on the other hand more components might be required (e.g (orchestrator)) to also handle annotations/meta-data locally, as well as to deal with identifiers and send all the relevant messages to the appropriate services etc.

The NeIC SDA is targeting both types of setup but also to allow for the possibility to re-use components in more use cases than initially envisioned.


### Container deployment options

The components of SDA are all container based using Docker standards for building container images. They can be deployed in a range of different ways depending on your local needs. The SDA developers are currently aware of the following alternatives in use:

1. Kubernetes (OpenShift)
2. Docker Swarm
3. PodMan

For testing on local developer PC's etc, Docker compose is part of the [local development and testing](guides/local-dev-and-testing.md) guide.


### Choice of storage back-end

To support different needs of different deployment locations, SDA is heavily configurable in several aspects. For the main archive storage, SDA supports both S3 storage and POSIX filesystem storage options, utilizing the exact same microservices with different parameters.

For other storage dependent functionality, such as upload areas i.e. `inbox` and data retrieval API, there are different choices of microservices (using different storage technology and transfer protocols) that can be orchestrated together with the main SDA microservices to meet local needs and requirements. 
