.. _`db`:

Database Setup
==============

We use a Postgres database (version 11.6+ ) to store intermediate data,
in order to track progress in file ingestion. The ``lega`` database
schema is documented below.

.. note:: Source code repository for DB component is available at: https://github.com/neicnordic/LocalEGA-db

The database container will initialize and create the necessary
database structure and functions if started with an empty area.
Procedures for *backing up the database* are important but considered
out of scope for the secure data archive project.
	  
Look at `the SQL definitions
<https://github.com/neicnordic/LocalEGA-db/tree/master/initdb.d>`_ if
you are also interested in the database triggers.

Configuration
-------------

The following environment variables can be used to configure the database:

+-----------------------------+-----------------------------------+---------------------+
|                Variable     | Description                       | Default value       |
+=============================+===================================+=====================+
|                ``PGVOLUME`` | Mountpoint for the writble volume | /var/lib/postgresql |
+-----------------------------+-----------------------------------+---------------------+
|  ``DB_LEGA_IN_PASSWORD``    | `lega_in`'s password              | -                   |
+-----------------------------+-----------------------------------+---------------------+
| ``DB_LEGA_OUT_PASSWORD``    | `lega_out`'s password             | -                   |
+-----------------------------+-----------------------------------+---------------------+
|                      ``TZ`` | Timezone for the Postgres server  | Europe/stockholm    |
+-----------------------------+-----------------------------------+---------------------+

For TLS support use the variables below:

+---------------------+--------------------------------------------------+-----------------------------------------------------------+
|         Variable    | Description                                      | Default value                                             |
+=====================+==================================================+===========================================================+
| ``PG_SERVER_CERT``  | Public Certificate in PEM format                 | `$PGVOLUME/pg.cert`                                       |
+---------------------+--------------------------------------------------+-----------------------------------------------------------+
|  ``PG_SERVER_KEY``  | Private Key in PEM format                        | `$PGVOLUME/pg.key`                                        |
+---------------------+--------------------------------------------------+-----------------------------------------------------------+
|           ``PG_CA`` | Public CA Certificate in PEM format              | `$PGVOLUME/CA.cert`                                       |
+---------------------+--------------------------------------------------+-----------------------------------------------------------+
| ``PG_VERIFY_PEER``  | Enforce client verification                      | 0                                                         |
+---------------------+--------------------------------------------------+-----------------------------------------------------------+
|        ``SSL_SUBJ`` | Subject for the self-signed certificate creation | `/C=SE/ST=Sweden/L=Uppsala/O=NBIS/OU=SysDevs/CN=LocalEGA` |
+---------------------+--------------------------------------------------+-----------------------------------------------------------+

.. note::  If not already injected, the files located at ``PG_SERVER_CERT``
           and ``PG_SERVER_KEY`` will be generated, as a self-signed public/private certificate pair, using ``SSL_SUBJ``.
           Client verification is enforced if and only if ``PG_CA`` exists and ``PG_VERIFY_PEER`` is set to ``1``.

Database schema
---------------

The current database schema is documented below.

Database schema migration
^^^^^^^^^^^^^^^^^^^^^^^^^

For continuity/ease of upgrade in production the database supports
automatic migrations between schema versions. This is handled by
migration scripts that each provide the migration from a specific
schema version to the next.

A schema version can contain multiple changes, but it is recommended
to group them logically. Some practical thinking is also useful - if
larger changes are required that risk being time consuming on large
databases, it may be best to split that work in small chunks.

Doing so helps in both demonstrating progress as well as avoiding
rollbacks of the entire process (and thus working needing to be done)
if something fails. Each schema migration is done in a transaction.

Schema versions are integers. There is no strong coupling between
releases of the secure data archive and database schema versions. A
new secure data archive release may increase several schema
versions/migrations or none.

.. IMPORTANT::
   Any changes done to database schema initialization should be
   reflected in a schema migration script.

Whenever you need to change the database schema, we recommended
changing both the database initialization scripts (and bumping the
bootstrapped schema version) as well as creating the corresponding
migration script to perform the changes on a database in use.

Migration scripts should be placed in `/migratedb.d/` in the
LocalEGA-db repo (https://github.com/neicnordic/LocalEGA-db). We
recommend naming them corresponding to the schema version they provide
migration to. There is an "eqmpty" migration script (`01.sql`) that can
be used as a template.

local_ega tables
^^^^^^^^^^^^^^^^

.. image:: /static/localega-schema.svg
   :alt: localega database schema

main
""""
This is the core table of the schema, which holds file identifiers, status, metadata, submission paths and checksum information, archive type information and cryptographic information.

+------------------------------------------+--------------------+
| Column Name                              | Data type          |
+==========================================+====================+
| archive_file_checksum                    | varchar            |
+------------------------------------------+--------------------+
| archive_file_checksum_type               | checksum_algorithm |
+------------------------------------------+--------------------+
| archive_file_reference                   | text               |
+------------------------------------------+--------------------+
| archive_file_size                        | int8               |
+------------------------------------------+--------------------+
| archive_file_type                        | storage            |
+------------------------------------------+--------------------+
| created_at                               | timestamptz        |
+------------------------------------------+--------------------+
| created_by                               | name               |
+------------------------------------------+--------------------+
| encryption_method                        | varchar            |
+------------------------------------------+--------------------+
| header                                   | text               |
+------------------------------------------+--------------------+
| id                                       | int4               |
+------------------------------------------+--------------------+
| last_modified                            | timestamptz        |
+------------------------------------------+--------------------+
| last_modified_by                         | name               |
+------------------------------------------+--------------------+
| stable_id                                | text               |
+------------------------------------------+--------------------+
| status                                   | varchar            |
+------------------------------------------+--------------------+
| submission_file_calculated_checksum      | varchar            |
+------------------------------------------+--------------------+
| submission_file_calculated_checksum_type | checksum_algorithm |
+------------------------------------------+--------------------+
| submission_file_extension                | varchar            |
+------------------------------------------+--------------------+
| submission_file_path                     | text               |
+------------------------------------------+--------------------+
| submission_file_size                     | int8               |
+------------------------------------------+--------------------+
| submission_user                          | text               |
+------------------------------------------+--------------------+
| version                                  | int4               |
+------------------------------------------+--------------------+

errors
""""""
This table keeps records of file submission errors, including information about the submitter and if the submission is active and also the hostname and the error type.

+-------------+-------------+
| Column Name | Data type   |
+=============+=============+
| active      | bool        |
+-------------+-------------+
| error_type  | text        |
+-------------+-------------+
| file_id     | int4        |
+-------------+-------------+
| from_user   | bool        |
+-------------+-------------+
| hostname    | text        |
+-------------+-------------+
| id          | int4        |
+-------------+-------------+
| msg         | text        |
+-------------+-------------+
| occurred_at | timestamptz |
+-------------+-------------+

session_key_checksums_sha256
""""""""""""""""""""""""""""
Checksums are recorded in order to keep track of already used session keys,

+---------------------------+--------------------+
| Column Name               | Data type          |
+===========================+====================+
| file_id                   | int4               |
+---------------------------+--------------------+
| session_key_checksum      | varchar            |
+---------------------------+--------------------+
| session_key_checksum_type | checksum_algorithm |
+---------------------------+--------------------+

status
""""""
This table holds file statuses, which can range from INIT, IN_INGESTION, ARCHIVED, COMPLETED, READY, ERROR and DISABLED.

+-------------+-----------+
| Column Name | Data type |
+=============+===========+
| code        | varchar   |
+-------------+-----------+
| description | text      |
+-------------+-----------+
| id          | int4      |
+-------------+-----------+

archive_encryption
""""""""""""""""""
It holds the cryptographic strategy used by the archive.

+-------------+-----------+
| Column Name | Data type |
+=============+===========+
| description | text      |
+-------------+-----------+
| mode        | varchar   |
+-------------+-----------+

local_ega views
^^^^^^^^^^^^^^^

archive_files
"""""""""""""

It contains all entries from the main table which are marked as ready.

errors
""""""

It contains error entries from active file submissions.

files
"""""

It mirrors the main table containing all records of submitted files.


local_ega functions
^^^^^^^^^^^^^^^^^^^

check_session_keys_checksums_sha256
"""""""""""""""""""""""""""""""""""
It returns if the session key checksums are already found in the database.

* Inputs: checksums

finalize_file
"""""""""""""
It flags files as READY, by setting their stable id and marking older ingestions as deprecated.

* Inputs: inbox_path, elixir_id, archive_file_checksum, archive_file_checksum_type, stable_id
* Target: local_ega.files

insert_error
""""""""""""
It adds an error entry of a file submission.

* Inputs: file_id, hostname, error_type, msg, from_user
* Target: local_ega.errors

insert_file
"""""""""""
It adds a new file entry and deprecates old faulty submissions of the same file if present.

* Inputs: submission_file_path, submission_user
* Target: local_ega.main

is_disabled
"""""""""""
It returns whether a given entry is disabled or not.

* Input: file id:

main_updated
""""""""""""
It synchronises the timestamp for each row after update on main.

* Input: None
* Target: local_ega.main

mark_ready
""""""""""
When triggered after a file is marked as READY, it deactivates all errors of the given entry.

* Inputs: None
* Target: mark_ready

local_ega_download tables
^^^^^^^^^^^^^^^^^^^^^^^^^

.. image:: /static/localega-download-schema.svg
   :alt: localega download database schema

requests
""""""""
It keeps track of all requests made to the file archive, including the requested file chunks and client information.

+------------------+-------------+
| Column Name      | Data type   |
+==================+=============+
| client_ip        | text        |
+------------------+-------------+
| created_at       | timestamptz |
+------------------+-------------+
| end_coordinate   | int8        |
+------------------+-------------+
| file_id          | int4        |
+------------------+-------------+
| id               | int4        |
+------------------+-------------+
| start_coordinate | int8        |
+------------------+-------------+
| user_info        | text        |
+------------------+-------------+

success
"""""""
A record of all successfully downloaded files.

+-------------+--------------+
| Column Name | Data type    |
+=============+==============+
| bytes       | int8         |
+-------------+--------------+
| id          | int4         |
+-------------+--------------+
| occurred_at | timestamptz  |
+-------------+--------------+
| req_id      | int4         |
+-------------+--------------+
| speed       | float8       |
+-------------+--------------+

errors
""""""
A record of all errors occurred during file requests, including the hostname and the error code.

+-------------+-------------+
| Column Name | Data type   |
+=============+=============+
| code        | text        |
+-------------+-------------+
| description | text        |
+-------------+-------------+
| hostname    | text        |
+-------------+-------------+
| id          | int4        |
+-------------+-------------+
| occurred_at | timestamptz |
+-------------+-------------+
| req_id      | int4        |
+-------------+-------------+

local_ega_download functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
download_complete
"""""""""""""""""
It marks a file download as complete, and calculates the download speed.
Inputs: requested file id, download size, speed
Target: local_ega_download.success

insert_error
""""""""""""

It adds an error entry of a file download.

* Inputs: requested file id, hostname, error code, error description
* Target: local_ega_download.errors

make_request
""""""""""""

It inserts a new request or reuses and old request entry of a given file.

* Inputs: stable id, user information, client ip, start coordinate and end coordinate
* Target: local_ega_download.requests

local_ega_ebi tables
^^^^^^^^^^^^^^^^^^^^

.. image:: /static/localega-ebi-schema.svg
   :alt: localega EBI database schema

filedataset
"""""""""""
It contains all entries that relate to EBI Files and Datasets.

+-------------------+-----------+
| Column Name       | Data type |
+===================+===========+
| dataset_stable_id | text      |
+-------------------+-----------+
| file_id           | int4      |
+-------------------+-----------+
| id                | int4      |
+-------------------+-----------+

fileindexfile
"""""""""""""
It contains all entries that relate to EBI Files and File indexes.

+----------------------+-----------+
| Column Name          | Data type |
+======================+===========+
| file_id              | int4      |
+----------------------+-----------+
| id                   | int4      |
+----------------------+-----------+
| index_file_id        | text      |
+----------------------+-----------+
| index_file_reference | text      |
+----------------------+-----------+
| index_file_type      | storage   |
+----------------------+-----------+

local_ega_ebi views
^^^^^^^^^^^^^^^^^^^^

file
""""
View for EBI Data-Out which contains all local_ega.main entries marked as ready.

file_dataset
""""""""""""
Used to synchronise with the entity eu.elixir.ega.ebi.downloader.domain.entity.FileDataset.

file_index_file
"""""""""""""""
Used to synchronise with the entity eu.elixir.ega.ebi.downloader.domain.entity.FileIndexFile.
