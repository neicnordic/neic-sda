sda-pipeline
============

Repository:
[neicnordic/sda-pipeline](https://github.com/neicnordic/sda-pipeline/)

`sda-pipeline` is part of
[NeIC Sensitive Data Archive](https://neic-sda.readthedocs.io/en/latest/) and
implements the components required for data submission.
It can be used as part of a [Federated EGA](https://ega-archive.org/federated)
or as a isolated Sensitive Data Archive. `sda-pipeline` was built with support
for both S3 and POSIX storage.

The SDA pipeline has four main steps:

1. [Ingest](ingest.md)
1. [Verify](verify.md)
1. [Finalize](finalize.md)
1. [Mapper](mapper.md)
