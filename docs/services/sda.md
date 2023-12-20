SDA - Sensitive Data Archive
============

Repository:
[neicnordic/sensitive-data-archive](https://github.com/neicnordic/sensitive-data-archive)

`sda` repository consists of a suite of services which are part of [NeIC Sensitive Data Archive](https://neic-sda.readthedocs.io/en/latest/) and implements the components required for data submission.
It can be used as part of a [Federated EGA](https://ega-archive.org/federated) or as a stand-alone (isolated) Sensitive Data Archive.
`sda` was built with support for both S3 and POSIX storage.

The SDA submission pipeline has four main steps:

1. [Ingest](ingest.md) splits file headers from files, moving the header to the database and the file content to the archive storage.
2. [Verify](verify.md) verifies that the header is encrypted with the correct key, and that the checksums match the user-provided checksums.
3. [Finalize](finalize.md) associates a stable accessionID with each archive file and backups the file.
4. [Mapper](mapper.md) maps file accessionIDs to a datasetID.

There are also additional support services:

1. [Intercept](intercept.md) relays messages from `CentralEGA` to the system.
2. [s3inbox](s3inbox.md) proxies uploads to the an S3 compatible storage backend.
3. [sync](sync.md) mirrors ingested data between sites in the [Bigpicture](https://bigpicture.eu/) project.
4. [syncapi](syncapi.md) is used in the [Bigpicture](https://bigpicture.eu/) project for mirroring data between two installations of SDA.
