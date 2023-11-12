# Troubleshooting

TODO:
This guide is a stub and has yet to be finished.
If you have feedback to give on the content you would like to see, please contact us on
[github](https://github.com/neicnordic/neic-sda)!

In this guide we aim to give some general tips on how to troubleshoot and restore services to working order.

## After deployment checklist

After having deployed the SDA services in a Federated setup, the following steps can be followed to ensure that everything is up and running correctly.

### Services running

The first step is to verify that the services are up and running and the credentials are valid. Make sure that,

- credentials for access to RabbitMQ and Postgres are securely injected to the respective services in the form of secrets
- all the pods/containers are in `Ready`/`Up` status.

Next step is to make sure that the remote connections (CEGA RabbitMQ) are working. Login to the RabbitMQ admin page and check that,

- the Federation status of the Admin tab is in state `running`
- the Shovel status of the Admin tab is in state `running` for all 5 shovels.

## End-to-end testing

NOTE: This guide assumes that there exists a test instance account with Central EGA. Make sure that the account is approved and added to the submitters group.

### Upload file(s)

Upload one or a number of files of different sizes and check that,

- the file(s) exists in the configured `inbox` of the storage backend (e.g. S3 bucket or POSIX path)
- the file(s) entry exists in the database in the `sda.files` and `sda.file_event_log` tables
- there exists an `uploaded` event for each specific file in the `sda.file_event_log`
- the file(s) exists in the CEGA metadata portal (here for the test instance) under the tab files which can be accessed after pressing the three lines button.

### Make a test submission

Make a submission with the portal and select the file(s) that were uploaded in the previous step. Once the analysis or runs (one of the two is required) step is finished, the messages for the ingestion of the files should appear in the logs of the `ingest` service. Make sure that,

- the messages are arriving for the file(s) included in the submission
- the `ingestion`, `verify` and `finalise` processes are started and send a message when finished
- the data in `sda.files` are correct
- the files are logged in the `sda.file_event_log` for each of the services and files
- the file(s) exists in the configured `archive` storage backend, see the `archive_file_path` in the `sda.files` table for the name of the archived file(s)
- the archived file(s) exists in the configured `backup` storage backend
- delete one run in the submitter portal, then and add it back again to make sure the cancel message is working as intended.

Finally, when all files have been ingested, the submission portal should allow for finalising the submission. The submission needs first to be accepted through a helpdesk portal. Once this step is done, make sure that,

- the message for the dataset arrives to the mapper service
- the dataset is created in the database and it includes the correct files by checking the `sda.datasets` and `sda.file_dataset` tables.

Once all the submission steps have been verified, we can assume that the pipeline part of the deployment is working properly.
