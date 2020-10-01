.. note:: Throughout this documentation, we can refer to `Central EGA <https://ega-archive.org/>`_ as
         ``CEGA``, or ``CentralEGA``, and *any* Local EGA (also known as Federated EGA) instance as ``LEGA``,
         or ``LocalEGA``.
         In the context of NeIC we will refer to the LocalEGA as the
         ``Sensitive Data Archive`` or ``SDA``.

===========================
NeIC Sensitive Data Archive
===========================

NeIC Sensitive Data Archive is divided into several microservices as illustrated
in the figure below.

.. figure:: https://docs.google.com/drawings/d/e/2PACX-1vQ5EMrwMb8X7efk_luHlkB1l1TEpTwuh-B2_c0SAoqPb5nSulKmt2cJj6ptt8oFFBHs2LLt8FHMl5VP/pub?w=960&h=540
   :width: 750
   :align: center
   :alt: General Architecture and Connected Components

The components/microservices can be classified by use case:

- submission - used in the process on submitting and ingesting data.
- data retrieval - used for data retrieval/download.

.. raw:: html
   :file: table.html


The overall data workflow consists of three parts:

- The users logs onto the Local EGA's inbox and uploads the
  encrypted files. They then go to the Central EGA's interface to prepare a
  submission;
- Upon submission completion, the files are ingested into the archive and
  become searchable by the Central EGA's engine;
- Once the file has been successfully archived, it can be accessed by
  researchers in accordance with permissions given by the
  corresponding Data Access Committee.

----

Central EGA contains a database of users with permissions to upload to
a specific Sensitive Data Archive. The Central EGA' ID is used to authenticate
the user against either their EGA password or a private key.

For every uploaded file, Central EGA receives a notification that the
file is present in a SDA's inbox.
The uploaded file must be encrypted in the :download:`Crypt4GH file format
<http://samtools.github.io/hts-specs/crypt4gh.pdf>` using that SDA public
Crypt4gh key.
The file is checksumed and presented in the Central
EGA's interface in order for the user to double-check that it was
properly uploaded.

More details about process in :ref:`inboxlogin`.

When a submission is ready, Central EGA triggers an ingestion process
on the user-chosen SDA instance. Central EGA's interface is updated with progress notifications
whether the ingestion was successful, or whether there was an error.

More details about the :ref:`ingestion process`.

Once a file has been successfully submitted and the ingestion process has been finalised,
including receiving an `Accession ID` from Central EGA. The Data Out API can be
utilised to retrieve set file by utilising the `Accession ID`. More details in :ref:`data out`.

----

Getting started
---------------

.. toctree::
   :maxdepth: 2
   :name: setup

   Getting started                   <setup>
   Database Setup                    <db>
   Deployments and Local Bootstrap   <deploy>

Information about the Architecture
----------------------------------

.. toctree::
   :maxdepth: 2
   :name: architecture

   Encryption                <encryption>
   Data Submission           <submission>
   Interfacing with CEGA     <connection>
   Data Retrieval API        <dataout>

Miscellaneous
-------------

.. toctree::
   :maxdepth: 1
   :name: extra

   Contributing              <https://github.com/neicnordic/neic-sda/blob/master/CONTRIBUTING.md>

| Version |version| | Generated |today|



.. |moreabout| unicode:: U+261E .. right pointing finger
.. |connect| unicode:: U+21cc .. <-_>
