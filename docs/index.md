
NeIC Sensitive Data Archive
===========================

The NeIC Sensitive Data Archive (SDA) is an encrypted data archive, implemented for storage of sensitive data. It is implemented as a modular microservice system that can be deployed in different configurations depending on the service needs.

The modular architecture of SDA supports both stand alone deployment of an archive, and the use case of deploying a Federated node in the [Federated European Genome-phenome Archive network (FEGA)](https://ega-archive.org/about/projects-and-funders/federated-ega/), serving discoverable sensitive datasets in the main [EGA web portal](https://ega-archive.org).

> NOTE:
> Throughout this documentation, we can refer to [Central
> EGA](https://ega-archive.org/) as `CEGA`, or `CentralEGA`, and *any*
> `FederatedEGA` instance also know as: `FEGA`, `LEGA` or
> `LocalEGA`. In the context of NeIC we will refer to the Federated EGA as the
> `Sensitive Data Archive` or `SDA`.


Organisation of the NeIC SDA Operations Handbook
------------------------------------------------

This operations handbook is organized in four  main parts, that each has it's own main section in the left navigation menu. Here we provide a condensed summary, follow the links below or use the menu navigation to each section's own detailed introduction page: 

1.  **Structure**: Provides overview material for how the services can be deployed in different constellations and highlights communication paths.

2.  **Communication**: Provides more detailed documentation focused on inter-service communication, as OpenAPI-specs for APIs, RabbitMQ message flow, and database information flow details.

3.  **Services**: Per service detailed specifications and documentation.

4.  **Guides**: Topic-guides for topics like _"Deployment"_, _"Federated vs. Stand-alone"_, _"Troubleshooting services"_, etc.


SDA Components and Architecture
-------------------------------

The main components and the interaction between them, based on the NeIC Sensitive Data Archive deployment in a `FederatedEGA` setup, are illustrated in the figure below. The different colored backgrounds represent different zones of separation in the federated deployment. 

![](https://docs.google.com/drawings/d/e/2PACX-1vSCqC49WJkBduQ5AJ1VdwFq-FJDDcMRVLaWQmvRBLy7YihKQImTi41WyeNruMyH1DdFqevQ9cgKtXEg/pub?w=960&h=540)

The components illustrated can be classified by which archive sub-process they take part in:

-   `Submission` - the process of submitting sensitive data and meta-data to the inbox staging area
-   `Ingestion` - the process of verifying uploaded data and securely storing it in archive storage, while synchronizing state and identifier information with CEGA
-   `Data Retrieval` - the process of re-encrypting and staging data for retrieval/download.


| Service/component                | Description                                                                                                                                                                              | Archive sub-process                      |
|----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------|
| Database                         | A Postgres database with appropriate schema, stores the file header, the accession id, file path and checksums as well as other relevant information.                                    | Submission, Ingestion and Data Retrieval |
| MQ                               | A RabbitMQ message broker with appropriate accounts, exchanges, queues and bindings. We use a federated queue to get messages from CentralEGA's broker and shovels to send answers back. | Submission and Ingestion                 |
| Inbox                            | Upload service for incoming data, acting as a dropbox. Uses credentials from `CentralEGA`.                                                                                               | Submission                               |
| Intercept                        | Relays messages between the queue provided from the federated service and local queues.                                                                                                  | Submission and Ingestion                 |
| [Ingest](services/ingest.md)     | Splits the Crypt4GH header and moves it to the database. The remainder of the file is sent to the storage backend (archive). No cryptographic tasks are done.                            | Ingestion                                |
| [Verify](services/verify.md)     | Using the archive crypt4gh secret key, this service can decrypt the stored files and checksum them against the embedded checksum for the unencrypted file.                               | Ingestion                                |
| [Finalize](services/finalize.md) | Handles the so-called Accession ID (stable ID) to filename mappings from CentralEGA.                                                                                                     | Ingestion                                |
| [Mapper](services/mapper.md)     | The mapper service register mapping of accessionIDs (stable ids for files) to datasetIDs.                                                                                                | Ingestion                                |
| Archive (Storage)                | Storage backend: can be a regular (POSIX) file system or a S3 object store.                                                                                                              | Ingestion and Data Retrieval             |
| [Data Retrieval API](dataout.md) | Provides a download/data access API for streaming archived data either in encrypted or decrypted format.                                                                                 | Data Retrieval                           |
| Inbox (Storage)                  | Storage backend: can be a regular (POSIX) file system or a S3 object store.                                                                                                              | Ingestion                                |
| Backup (Storage)                 | Storage backend: can be a regular (POSIX) file system or a S3 object store.                                                                                                              | Ingestion                                |

------------------------------------------------------------------------
