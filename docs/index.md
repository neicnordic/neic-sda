> NOTE:
> Throughout this documentation, we can refer to [Central
> EGA](https://ega-archive.org/) as `CEGA`, or `CentralEGA`, and *any*
> Local EGA (also known as Federated EGA) instance as `LEGA`, or
> `LocalEGA`. In the context of NeIC we will refer to the LocalEGA as the
> `Sensitive Data Archive` or `SDA`.

NeIC Sensitive Data Archive
===========================

NeIC Sensitive Data Archive is divided into several microservices as
illustrated in the figure below.

![](https://docs.google.com/drawings/d/e/2PACX-1vSCqC49WJkBduQ5AJ1VdwFq-FJDDcMRVLaWQmvRBLy7YihKQImTi41WyeNruMyH1DdFqevQ9cgKtXEg/pub?w=1440&amp;h=810)

The components/microservices can be classified by use case:

-   submission - used in the process on submitting and ingesting data.
-   data retrieval - used for data retrieval/download.



Service | Description | Use cases activating service | Status
-------:|:------------|:-----------------------------|:-----:
db | A Postgres database with appropriate schema, stores the file header the accession id, file path and checksums as well as other relevant information. | Submission and Data Retrieval | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
mq (broker) | A RabbitMQ message broker with appropriate accounts, exchanges, queues and bindings. We use a federated queue to get messages from CentralEGA's broker and shovels to send answers back.| Submission | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
Inbox | Upload service for incoming data, acting as a dropbox. Uses credentials from Central EGA. | Submission | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
Intercept | relays message between the queue provided from the federated service and local queues. | Submission | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
Ingest | Splits the Crypt4GH header and moves it to database. The remainder of the file is sent to the storage backend (archive). No cryptographic tasks are done. | Submission | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
Verify | Uses a crypt4gh secret key, this service can decrypt the stored files and checksum them against the embedded checksum for the unencrypted file. | Submission | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
Archive | Storage backend: can be a regular (POSIX) file system or a S3 object store. | Submission and Data Retrieval | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
Finalize | Handles the so-called <i>Accession ID (stable ID)</i> to filename mappings from CentralEGA store. | Submission | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
Mapper | The mapper service register mapping of accessionIDs (stable ids for files) to datasetIDs. | Submission Data Retrieval | <i class="fa fa-battery-full ega-stable" title="Stable"></i>
Data Out API | Provides a download/data access API for streaming archived data either in encrypted or decrypted format. | Data Retrieval | <i class="fa fa-battery-half ega-dev" title="Work in progress"></i>
Metadata | Component used in standalone version of SDA. Provides an interface and backend to submit Metadata and associated with a file in the Archive. | Submission Data Retrieval | <i class="fa fa-battery-half ega-dev" title="Work in progress"></i>
Orchestrator | Component used in standalone version of SDA. Provides an automated ingestion and dataset ID and file ID mapping. | Submission Data Retrieval | <i class="fa fa-battery-half ega-dev" title="Work in progress"></i>


The overall data workflow consists of three parts:

-   The users logs onto the Local EGA's inbox and uploads the encrypted
    files. They then go to the Central EGA's interface to prepare a
    submission;
-   Upon submission completion, the files are ingested into the archive
    and become searchable by the Central EGA's engine;
-   Once the file has been successfully archived, it can be accessed by
    researchers in accordance with permissions given by the
    corresponding Data Access Committee.

------------------------------------------------------------------------

Central EGA contains a database of users with permissions to upload to a
specific Sensitive Data Archive. The Central EGA ID is used to
authenticate the user against either their EGA password or a private
key.

For every uploaded file, Central EGA receives a notification that the
file is present in a SDA's inbox. The uploaded file must be encrypted
in the [Crypt4GH file format](http://samtools.github.io/hts-specs/crypt4gh.pdf) using that SDA public Crypt4gh key. The file is
checksumed and presented in the Central EGA's interface in order for
the user to double-check that it was properly uploaded.

More details about process in [Data Submission](submission.md#data-submission).

When a submission is ready, Central EGA triggers an ingestion process on
the user-chosen SDA instance. Central EGA's interface is updated with
progress notifications whether the ingestion was successful, or whether
there was an error.

More details about the [Ingestion Workflow](submission.md#ingestion-workflow).

Once a file has been successfully submitted and the ingestion process
has been finalised, including receiving an `Accession ID` from Central
EGA. The Data Out API can be utilised to retrieve set file by utilising
the `Accession ID`. More details in [Data Retrieval API](dataout.md#data-retrieval-api).

------------------------------------------------------------------------
