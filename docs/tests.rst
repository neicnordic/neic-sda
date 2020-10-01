Testing
-------

We have implemented 2 types of testsuite: 

- one set of *unit tests* to test the functionalities of the code; 
- one set of *integration tests* to test the overall architecture.

The latter does actually deploy a chosen setup and runs several scenarios, users
will utilize the system as a whole, .

.. note:: Unit tests and integration tests are automatically executed 
          with every push and PR to the ``NBIS Github repo``.


Unit Tests
^^^^^^^^^^

Unit tests are minimal: Given a set of input values for a chosen
function, they execute the function and check if the output has the
expected values. Moreover, they capture triggered exceptions and
errors. Additionally we also perform PEP8 and PEP257 coding style guide checks
using `flake8  <http://flake8.pycqa.org/en/latest/>`_.
Sphinx documentation check for links consistency and HTML output
and `tox <http://tox.readthedocs.io/>`_ to automate running tests. 

Unit tests can in parellel be run using the ``tox`` commands.

.. code-block:: console

    $ cd [git-repo]
    $ tox -p auto

Integration Tests
^^^^^^^^^^^^^^^^^

Unit Tests are run with pytest, coverage and tox.
The other tests use `BATS <https://github.com/bats-core/bats-core>`_.

Integration Scenarios
"""""""""""""""""""""

These tests treat the system as a black box, only checking the expected output for a given input.

- [x] Ingesting a 10MB file - Expected outcome: Message in the CentralEGA completed queue
- [x] Ingesting a 1 GB file - Expected outcome: Message in the CentralEGA completed queue
- [x] Ingesting a directory with multiple files and/or subdirectories - Expected outcome: Messages in the CentralEGA completed queue
- [x] Upload 2 files encrypted with same session key - Expected outcome: Message in the CentralEGA user-error queue for the second file
- [x] Upload 2 files with the same name (with ingestion in-between)
- [x] (skipped) Use 2 stable IDs for the same ingested file - Expected outcome: Error captured
- [x] Ingest a file with a user that does not exist in CentralEGA - Expected outcome: Authentication "fails", and the ingestion does not start
- [x] Ingest a file with a user in CentralEGA, using the wrong password - Expected outcome: Authentication "fails", and the ingestion does not start
- [x] Ingest a file with a user in CentralEGA, using the wrong sshkey - Expected outcome: Fallback to password (previous scenario)
- [x] Ingest a file for a given LocalEGA using the key of another one - Expected outcome: Message in the CentralEGA error queue, with the relevant content.
- [x] Ingestion with wrong file format - Expected outcome: Message in the CentralEGA error queue, with the relevant content.
- [x] Receiving an accession ID - Expected outcome: Accession ID is present in the database

Robustness Scenarios
""""""""""""""""""""

These tests will not treat the system as a black box.
They require some knowledge on how the components are interconnected.

- [ ] Check Archive+DB consistency - Expected outcome: Re-checksums the files after several ingestions
- [x] (skipped) DB restarted after *n* seconds - Expected outcome: Combining an ingestion before and one after, the latest one should still "work"
- [x] (skipped) DB restarted in the middle of an ingestion - Expected outcome: File ingested as usual
- [x] (skipped) MQ restarted, test delivery mode -Expected outcome: queued tasks completed
- [ ] Retry message 3 times if rejected before error or timeout - Expected outcome: queued tasks completed
- [x] (skipped) Restart some component X - Expected outcome: Business as usual

Stress Scenarios
""""""""""""""""

These tests treat the system as a black box and "measure" performance

- [ ] Multiple ingestions by the same user
- [ ] Ingestions by multiple users
- [ ] (Auto?)-Scaling
  
Security Scenarios
""""""""""""""""""

These tests will not treat the system as a black box.
They require some knowledge on how the components are interconnected.

- [ ] Network access forbidden from some selected components
- [x] Inbox user isolation: A user cannot access the files of another user -Expected outcome: File not found or access denied
