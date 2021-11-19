Interfacing with CEGA |connect| SDA
===================================

All Local EGA instances are connected to Central EGA using
`RabbitMQ`_, a Message Broker, that allows the components to
send and receive messages, which are queued, not lost, and resent
on network failure or connection problems.

The RabbitMQ message brokers of each SDA instance are the **only**
components with the necessary credentials to connect to Central EGA
message broker.

We call ``CEGAMQ`` and ``LocalMQ`` (Local Message Broker, also known as ``sda-mq``),
the RabbitMQ message brokers of, respectively, ``Central EGA``
and ``SDA``/``LocalEGA``.

.. _`mq`:

Local Message Broker
--------------------

.. note:: Source code repository for MQ component is available at: https://github.com/neicnordic/sda-mq


Configuration
^^^^^^^^^^^^^

The following environment variables can be used to configure the broker:

.. note:: We use `RabbitMQ 3.7.8`_ including the management plugins.

+----------------------+----------------------------------------------+
| Variable             | Description                                  |
+======================+==============================================+
| ``MQ_VHOST``         | Default vhost other than ``/``               |
+----------------------+----------------------------------------------+
| ``MQ_VERIFY``        | Set to ``verify_none`` to disable            |
|                      | verification of client certificate           |
+----------------------+----------------------------------------------+
| ``MQ_USER``          | Default user (with admin rights)             |
+----------------------+----------------------------------------------+
| ``MQ_PASSWORD_HASH`` | Password hash for the above user             |
+----------------------+----------------------------------------------+
| ``CEGA_CONNECTION``  | DSN URL for the shovels and federated queues |
|                      | with CentralEGA                              |
+----------------------+----------------------------------------------+


.. note:: For SDA stand-alone do not use ``CEGA_CONNECTION`` and do not set up
          ``Intercept`` service. This will cause no messages to be shoveled to a
          CentralEGA, whilst the queues stay the same. ``Orchestrator`` service
          would need to be set up to send and recive messages between other services.

Central EGA connection
----------------------

``CEGAMQ`` declares a ``vhost`` for each SDA instance. It also
creates the credentials to connect to that ``vhost`` in the form of a
*username/password* pair. The connection uses the AMQP(S) protocol.

``LocalMQ`` then uses a connection string with the following syntax:

.. code-block:: console

   amqp[s]://<user>:<password>@<cega-host>:<port>/<vhost>

.. note:: All the messages received from CEGA are intercepted by ``Intercept`` service
          and forwarded to the right queue in the ``LocalMQ``


``CEGAMQ`` contains an exchange named ``localega.v1``. ``v1`` is used for
versioning and is internal to CentralEGA. The queues connected to that
exchange are also internal to CentralEGA.

+-----------------+-------------------------------------------------+
| Name            | Purpose                                         |
+=================+=================================================+
| files           | Triggers for file ingestion                     |
+-----------------+-------------------------------------------------+
| completed       | When files are backed up                        |
+-----------------+-------------------------------------------------+
| verified        | When files are properly ingested  and verified  |
+-----------------+-------------------------------------------------+
| errors          | User-related errors                             |
+-----------------+-------------------------------------------------+
| inbox           | Notifications of uploaded files                 |
+-----------------+-------------------------------------------------+

``LocalMQ`` contains two exchanges named ``sda`` and ``to_cega``,
and the following queues, in the default ``vhost``:

+-----------------+---------------------------------------+
| Name            | Purpose                               |
+=================+=======================================+
| archived        | Archived files                        |
+-----------------+---------------------------------------+
| backup          | Signal files to backup                |
+-----------------+---------------------------------------+
| completed       | Files are backed up                   |
+-----------------+---------------------------------------+
| error           | User-related errors                   |
+-----------------+---------------------------------------+
| files           | Receive notification for ingestion    |
|                 | from  ``CEGAMQ`` or Orchestrator      |
+-----------------+---------------------------------------+
| inbox           | Notifications of uploaded files       |
+-----------------+---------------------------------------+
| ingest          | Trigger for file ingestion            |
+-----------------+---------------------------------------+
| mappings        | Received Dataset to file mapping      |
+-----------------+---------------------------------------+
| accessionIDs    | Receive Accession IDs from ``CEGAMQ`` |
|                 | or Orchestrator                       |
+-----------------+---------------------------------------+
| verified        | Files ingested and verified           |
+-----------------+---------------------------------------+

``LocalMQ`` registers ``CEGAMQ`` as an *upstream* and listens to the
incoming messages in ``files`` using a *federated queue*.  Ingestion
workers listen to the ``files`` queue of the local broker. If there
are no messages to work on, ``LocalMQ`` will ask its upstream queue if
it has messages. If so, messages are moved downstream. If not the
Ingest Service will wait for messages to arrive.

.. note:: More information can be found also at: 
          https://localega.readthedocs.io/en/latest/amqp.html#message-interface-api-cega-connect-lega


``CEGAMQ`` receives notifications from ``LocalMQ`` using a
*shovel*. Everything that is published to its ``to_cega`` exchange gets
forwarded to CentralEGA (using the routing key based on the name ``files.<internal_queue_name>``).
We propagate the different status of the workflow to CentralEGA, using
the following routing keys:

+-----------------------+-------------------------------------------------------+
| Name                  | Purpose                                               |
+=======================+=======================================================+
| files.completed       | For back-up files, ready to be distributed            |
+-----------------------+-------------------------------------------------------+
| files.error           | In case a user-related error is detected              |
+-----------------------+-------------------------------------------------------+
| files.inbox           | For inbox file operations                             |
+-----------------------+-------------------------------------------------------+
| files.verified        | For files ready to request accessionID                |
+-----------------------+-------------------------------------------------------+

Note that we do not need at the moment a queue to store the completed
message, nor the errors, as we forward them to Central EGA.


.. image:: /static/CEGA-LEGA.png
   :alt: RabbitMQ setup

.. _supported checksum algorithm: md5

Connecting SDA to Central EGA
-----------------------------

Central EGA only has to prepare a user/password pair along with a
``vhost`` in their RabbitMQ.

When Central EGA has communicated these details to the given Local EGA
instance, the latter can contact Central EGA using the federated queue
and the shovel mechanism in their local broker.

CentralEGA should then see 2 incoming connections from that new
LocalEGA instance, on the given ``vhost``.

The exchanges and routing keys will be the same as all the other
LocalEGA instances, since the clustering is done per ``vhost``.

.. _`message`:

Message Format
^^^^^^^^^^^^^^

It is necessary to agree on the format of the messages exchanged
between Central EGA and any Local EGAs. Central EGA's messages are
JSON-formatted.

The JSON schemas can be found in: https://github.com/neicnordic/sda-pipeline/tree/master/schemas 

When a ``Submission Inbox`` sends a message to CentralEGA it contains the
following:

.. code-block:: javascript

   {
      "operation": "upload",
      "user":"john",
      "filepath":"somedir/encrypted.file.gpg",
      "encrypted_checksums": [
         { "type": "md5", "value": "abcdefghijklmnopqrstuvwxyz"},
         { "type": "sha256", "value": "12345678901234567890"}
      ]
   }

In order to identify the type of inbox activity,
``operation`` in the above message can have the following values:

* ``upload`` - when a file is uploaded;
* ``remove`` - when a file is deleted;
* ``rename`` - when a file is renamed.

CentralEGA triggers the ingestion and the message sent to ``files`` queue
contains the same information. In order to distinguish messages,
Central EGA adds a field named type to all outgoing messages. 
There are 5 types of messages:

* ``type=ingest``: an ingestion trigger
* ``type=cancel``: an ingestion cancellation
* ``type=accession``: contains an accession id
* ``type=mapping``: contains a dataset to accession ids mapping
* ``type=heartbeat``: A mean to check if the Local EGA instance is “alive”


.. important:: The ``encrypted_checksums`` key is optional. If the key is not present
               the sha256 checksum will be calculated by ``Ingest`` service.


The ``Ingest`` service upon successful operation will send a message to
``archived`` queue containing:

.. code-block:: javascript

   {
      "user":"john",
      "fileID": "1",
      "filepath":"somedir/encrypted.file.c4gh",
      "archivePath": "somedir/archived.file.c4gh",
      "encrypted_checksums": [
         { "type": "sha256", "value": "12345678901234567890"}
      ]
   }

``Verify`` service will consume set message and will forward to ``verified`` queue
and *shoveled* to ``CEGAMQ`` but also adding a key ``decrypted_checksums``, 
which will respond with the same content, but adding the `Accession ID`.

.. code-block:: javascript
   
   {
      "user":"john",
      "filepath":"somedir/encrypted.file.c4gh",
      "decrypted_checksums": [
         { "type": "md5", "value": "abcdefghijklmnopqrstuvwxyz"},
         { "type": "sha256", "value": "12345678901234567890"}
      ]
   }

``Finalize`` service should receive the message below and assign the `Accession ID` to the
corresponding file and send a message to ``backup`` queue for the backup services or in case there 
is no backup service to the ``completed`` queue.

.. code-block:: javascript

   {
      "user":"john",
      "filepath":"somedir/encrypted.file.c4gh",
      "accession_id": "EGAF12345678901",
      "decrypted_checksums": [
         { "type": "md5", "value": "abcdefghijklmnopqrstuvwxyz"},
         { "type": "sha256", "value": "12345678901234567890"}
      ]
   }


``Mapper`` service after the file has been published should receive a message
containing accession IDs mapping between files and datasets

.. code-block:: javascript

   {
      "user":"john",
      "dataset_id": "EGAD12345678901",
      "accession_ids": ["EGAF12345678901", "EGAF12345678902"]
   }


.. |connect| unicode:: U+21cc .. <->
.. _RabbitMQ: http://www.rabbitmq.com
.. _RabbitMQ 3.7.8: https://hub.docker.com/_/rabbitmq
