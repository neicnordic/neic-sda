Installation
============

.. highlight:: shell

The sources for SDA can be downloaded and installed from the `NeIC Github repo`_.

.. code-block:: console

    $ git clone https://github.com/neicnordic/sda-pipeline.git
    $ go build

The recommended method is however to use one of our deployment
strategies: 
    
- `Kubernetes Helm charts <https://github.com/neicnordic/sda-helm/>`_;
- `Docker Swarm <https://github.com/neicnordic/LocalEGA-deploy-swarm/>`_.

Configuration
-------------

Starting the SDA submission services require a running Database and Message Broker,
the setup for those components is detailed in:

- :ref:`db`;
- :ref:`mq`.

:ref:`data out` requires a working Database in order to be set up.

A few files are required in order to connect the different components.

The main configurations are set by default, and it is possible to
overwrite any of them. The microservices can be started
using the ``--conf <file>`` switch to specify the configuration file.

The settings are loaded in the following order:

* from environment variables (where the naming convention is uppercase 
  ``section_option`` (as in ``default.ini``), e.g. ``ARCHIVE_STORAGE_DRIVER`` or ``POSTGRES_DB``,
* from the package's ``defaults.ini``,
* from the file ``/etc/ega/conf.ini`` (if it exists),
* and finally from the file specified as the ``--conf`` argument.

Therefore, there is no need to update the ``defaults.ini``. Instead,
reset/update any key/value pairs by creating a custom configuration file and pass it
to ``--conf`` as a command-line argument.

Default configuration file contains the following information:

.. literalinclude:: /../lega/conf/defaults.ini
   :language: python

.. _NeIC Github repo: https://github.com/neicnordic/sda-pipeline
