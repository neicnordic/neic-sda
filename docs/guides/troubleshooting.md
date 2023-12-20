# Troubleshooting

In this guide we aim to give some general tips on how to troubleshoot and restore services to working order.

## After deployment checklist

After having deployed the SDA services in a `FederatedEGA` setup, the following steps can be followed to ensure that everything is up and running correctly.

### Services running

The first step is to verify that the services are up and running and the credentials are valid. Make sure that:

- credentials for access to RabbitMQ and Postgres are securely injected to the respective services in the form of secrets
- all the pods/containers are in `Ready`/`Up` status and and no restarts among the pods/containers.
    - for `FederatedEGA` setup the following pods are required: `intercept`, `ingest`, `verify`, `finalize`, `mapper` and a [Data Retrieval API](/docs/dataout.md)
    - check the pods/container logs contain as the last message (after they have started): `time="2023-12-12T19:25:02Z" level=info msg="Starting <service-name> service"` for `intercept`, `ingest`, `verify`, `finalize`, `mapper`
    - check the pods/container logs contain as the last message (after they have started) `time="2023-12-12T19:28:16Z" level=info msg="(5/5) Starting web server"` for `download` Data Retrieval service

Next step is to make sure that the remote connections (`CentralEGA` RabbitMQ) are working. Login to the RabbitMQ admin page and check that:

- the [Federation](https://www.rabbitmq.com/federation.html) status of the Admin tab is in state `running`
  or using `rabbitmqctl federation_status` from the command line of a RabbitMQ pod/container.
- the [Shovel](https://www.rabbitmq.com/shovel.html) status of the Admin tab is in state `running` for all shovels 
  or using `rabbitmqctl shovel_status` from the command line of a RabbitMQ pod/container.

## End-to-end testing

> NOTE: 
> This guide assumes that there exists a test instance account with `CentralEGA`. Make sure that the account is approved and added to the submitters group.
> The [local development and testing](local-dev-and-testing.md) guide provides the scripts for testing different parts of the setup, that can be used
> as a reference.

### Upload file(s)

Upload one or a number of files of different sizes and check that,

- the file(s) exists in the configured `inbox` of the storage backend (e.g. S3 bucket or POSIX path)
- the file(s) entry exists in the database in the `sda.files` and `sda.file_event_log` tables
- If the `s3inbox` is used, there should be an `uploaded` event for each specific file in the `sda.file_event_log`
- the file(s) exists in the `CentralEGA` [Submission portal](https://ega-archive.org/submission/metadata/submission/sequencing-phenotype/submitter-portal/) (the submission portal URL address is specific for each `FederatedEGA` node). `Files` listing, which can be accessed after pressing the three lines menu button.

### Make a test submission

Make a submission with the portal and select the file(s) that were uploaded in the previous step. Once the analysis or runs (one of the two is required) step is finished, the messages for the ingestion of the files should appear in the logs of the `ingest` service. Make sure that:

- the messages are arriving for the file(s) included in the submission
- the `ingestion`, `verify` and `finalize` processes are started and send a message when finished
- the data in `sda.files` table are correct
- the files are logged in the `sda.file_event_log` table for each of the services and files
- the file(s) exists in the configured `archive` storage backend, see the `archive_file_path` in the `sda.files` table for the name of the archived file(s)
- the archived file(s) exists in the configured `backup` storage backend
- delete one run in the submitter portal, then and add it back again to make sure the cancel message is working as intended.

Finally, when all files have been ingested, the submission portal should allow for finalising the submission. The submission needs first to be accepted through a helpdesk portal. Once this step is done, make sure that,

- the message for the dataset arrives to the mapper service
- the dataset is created in the database and it includes the correct files by checking the `sda.datasets` and `sda.file_dataset` tables.
- the dataset has the status `registered` in the `sda.dataset_event_log`
- the dataset gets the status `released` in the `sda.dataset_event_log`, this might take a while depending on what date was chosen in the submitter portal.

Once all the submission steps have been verified, we can assume that the pipeline part of the deployment is working properly.
