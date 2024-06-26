Data Submission
===============

Ingestion Procedure
-------------------

For a given `FederatedEGA` node, `CentralEGA` selects the associated `vhost` and
drops, in the `files` queue, one message per file to ingest.

Structure of the message and its contents are described in
[Message Format](connection.md#message-format).

> NOTE:
> Source code repository for Submission components is available at:
> <https://github.com/neicnordic/sensitive-data-archive>

### Ingestion Workflow

```mermaid
   
   sequenceDiagram
      autonumber
      participant Upload Tool
      box SDA
      participant Inbox
      participant Ingest
      participant Verify
      participant Finalize
      participant Mapper
      participant SDA Database
      participant Intercept
      participant SDA RabbitMQ
      end
      box Central EGA
      participant Central EGA RabbitMQ
      end
      Upload Tool->>Inbox: upload encrypted file
      activate Inbox
      Inbox-->>SDA RabbitMQ: msg: Upload Done
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.inbox]
      deactivate Inbox
      Central EGA RabbitMQ-->>SDA RabbitMQ: federated msg: [from_cega][ingest type]
      SDA RabbitMQ-->>Intercept: Intercept reads message
      Intercept -->> SDA RabbitMQ: Forwards ingest message <br/> to queue
      alt Ingest is successful 
      SDA RabbitMQ->>Ingest: msg: [sda][ingest] begin ingestion
      activate Ingest 
      Ingest->>SDA Database: mark ingested
      Note over Ingest: store file in Archive
      Ingest->>SDA Database: mark archived
      Ingest-->>SDA RabbitMQ: msg [sda][archived]
      else Error occurred in ingestion process
      Ingest-->>SDA RabbitMQ: msg: error
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.error]
      end
      deactivate Ingest
      alt Verify is successful 
      activate Verify
      SDA RabbitMQ-->>Verify: msg [sda][archived] triggers verify
      Verify->>SDA Database: mark verified
      Verify-->>SDA RabbitMQ: msg: [sda][verified]
      else Error occurred in verify process
      Verify-->>SDA RabbitMQ: msg: error
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.error]
      end
      deactivate Verify
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.verified]
      Central EGA RabbitMQ-->>SDA RabbitMQ: federated msg: [from_cega][accession type]
      SDA RabbitMQ-->>Intercept: Intercept reads message
      Intercept -->> SDA RabbitMQ: Forwards accession ID message <br/> to queue
      SDA RabbitMQ->>Finalize: msg: [sda][accession] map file to accession ID
      alt Finalize is successful 
      activate Finalize
      note right of Finalize: Finalize makes the file backup
      Finalize->>SDA Database: mark completed
      Finalize-->>SDA RabbitMQ: msg: [sda][completed]
      else Error occurred in finalize process
      Finalize-->>SDA RabbitMQ: msg: error
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.error]
      end
      deactivate Finalize
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.completed]
      Central EGA RabbitMQ-->>SDA RabbitMQ: federated msg: [from_cega][mappings type]
      SDA RabbitMQ-->>Intercept: Intercept reads message
      Intercept -->> SDA RabbitMQ: Forwards mapper message of type mapping <br/> to queue
      SDA RabbitMQ->>Mapper: msg: [sda][mappings] map dataset to file accession ID
      alt Mapper Mapper creates dataset ID to file accession ID mapping 
      activate Mapper
      Mapper->>SDA Database: map file to dataset accession ID
      Mapper->>Inbox: remove file from inbox
      else Error occurred in mapper process
      Mapper-->>SDA RabbitMQ: msg: error
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.error]
      end
      Central EGA RabbitMQ-->>SDA RabbitMQ: federated msg: [from_cega][release type]
      SDA RabbitMQ-->>Intercept: Intercept reads message
      Intercept -->> SDA RabbitMQ: Forwards mapper message <br/> to queue
      SDA RabbitMQ->>Mapper: msg: [sda][mappings] release dataset
      alt Mapper flags dataset ready for release 
      activate Mapper
      Mapper->>SDA Database: flag dataset ready for release
      else Error occurred in mapper process
      Mapper-->>SDA RabbitMQ: msg: error
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.error]
      end
      Central EGA RabbitMQ-->>SDA RabbitMQ: federated msg: [from_cega][deprecate type]
      SDA RabbitMQ-->>Intercept: Intercept reads message
      Intercept -->> SDA RabbitMQ: Forwards mapper message  <br/> to queue
      SDA RabbitMQ->>Mapper: msg: [sda][mappings] deprecate dataset
      alt Mapper flags dataset as deprecated 
      activate Mapper
      Mapper->>SDA Database: flag dataset as deprecated
      else Error occurred in mapper process
      Mapper-->>SDA RabbitMQ: msg: error
      SDA RabbitMQ-->>Central EGA RabbitMQ: shovel msg:[to_cega][files.error]
      end
      deactivate Mapper

```

> NOTE:
> Ingestion Workflow Legend
>
> The sequence diagram describes the different phases during the ingestion
> process. The elements at the top represent each of the services or
> actuators involved in the workflow. The interaction between these is
> depicted by horizontal arrows connecting the elements.
>
> The vertical axis represents time progression down the page, where
> processes are marked with colored vertical bars. The colors used for the
> services/actuators match those used for the events initiated by the
> respective services, except for the interactions in case of errors,
> which are highlighted with red. The optional fragments are only executed
> if errors occur in `ingest`, `verify` or `finalize` services. 
> **Note that the time axis in this diagram is all about the sequence of events not duration.**

### Ingestion Steps

The `Ingest` service (can be replicated) reads file from the
`Submission Inbox` and splits Crypt4GH header from the beginning of the
file, puts it in a database and sends the remainder to the `Archive`,
leveraging the Crypt4GH format.

> NOTE:
> There is no decryption key retrieved during that step. The `Archive` can
> be either a regular file system on disk, or an S3 object storage.
> `Submission Inbox` can also have as a backend a regular file system or
> S3 object storage.

The files are read chunk by chunk in order to bound the memory usage.
After completion, a message is dropped into the local message broker to
signal that the `Verify` service can check the file corresponds to what
was submitted. It also ensures that the stored file is decryptable and
that the integrated checksum is valid.

At this stage, the associated decryption key is retrieved. If decryption
completes and the checksum is valid, a message of completion is sent to
`CentralEGA`: Ingestion completed.

> Important:
> If a file disappears or is overwritten in the inbox before ingestion is completed, ingestion may not be possible.

Should any of the aforementioned steps result in an error, the workflow is terminated, and the error is logged. If the error is attributed to user misuse, such as providing an incorrect checksum or tampering with the encrypted file, it is reported to `CentralEGA` for display in the Submission Interface.


Submission Inbox
----------------

`CentralEGA` contains a database of users, with IDs and passwords. Multiple solutions
have been developed to facilitate user authentication 
against the CentralEGA user database.:

- [Apache Mina Inbox](submission.md##sftp-inbox);
- [S3 Proxy Inbox](submission.md#s3-proxy-inbox);
- [TSD File API](submission.md#tsd-file-api).

Every solution utilizes CentralEGA's user IDs and is planned for
extension to incorporate Elixir IDs, from which the `@elixir-europe.org` suffix is removed.

The procedure is as follows: the inbox is started without any created
user. When a user wants to log into the inbox (via `sftp`, `s3` or
`https`), the inbox service looks up the username in a local queries the
CentralEGA REST endpoint. Upon the user's return, their credentials are 
stored in the local cache, and a home directory for the user is created. 
The user now gets logged in if the password or public key authentication succeeds.

{%
   include-markdown "services/sftpinbox.md"
   heading-offset=3
%}

> NOTE:
> Sources are located at the separate repository:
> <https://github.com/neicnordic/sensitive-data-archive/tree/main/sda-sftp-inbox> Essentially, it's a
> Spring-based Maven project, integrated with the
> [Local Message Broker](connection.md#local-message-broker).


### TSD File API

In order to utilise Tryggve2 SDA within
[TSD](https://www.uio.no/english/services/it/research/sensitive-data/)
Several components have been developed:

-   <https://github.com/unioslo/tsd-file-api>
-   <https://github.com/uio-bmi/LocalEGA-TSD-proxy>
-   <https://github.com/unioslo/tsd-api-client>

>NOTE:
> Access is restricted to UiO network. Please, contact TSD support for the
> access, if needed. Documentation:
> <https://test.api.tsd.usit.no/v1/docs/tsd-api-integration.html>


### S3 Proxy Inbox

> NOTE:
> Sources are located at the separate repository:
> <https://github.com/neicnordic/sensitive-data-archive/blob/main/sda/cmd/s3inbox/>

The S3 Proxy uses access tokens as the main authentication mechanism.

The sda authentication service
(<https://github.com/neicnordic/sensitive-data-archive/tree/main/sda-auth>) is designed to convert CEGA
REST endpoint authentication to a JWT that can be used when uploading to
the S3 proxy.

The proxy requires the user to set the bucket name the same as the
username when uploading data,
`s3cmd put FILE s3://USER_NAME/path/to/file`


{%
   include-markdown "services/s3inbox.md"
   heading-offset=3
%}
