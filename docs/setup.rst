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

.. _NeIC Github repo: https://github.com/neicnordic/sda-pipeline
