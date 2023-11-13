Installation
============

The sources for SDA can be downloaded and installed from the [NeIC
Github repo](https://github.com/neicnordic/sensitive-data-archive).

In order to build binaries:
```bash
$ git clone https://github.com/neicnordic/sensitive-data-archive.git
$ cd sda
$ for p in cmd/*; do go build -buildvcs=false -o "${p/cmd\//sda-}" "./$p"; done
```

To be able to develop the source code
```bash
$ git clone https://github.com/neicnordic/sensitive-data-archive.git
$ go work init
$ go work use ./sda
$ cd sda
```

The recommended method is however to use one of our deployment
strategies:

-   [Kubernetes Helm charts](https://github.com/neicnordic/sensitive-data-archive/tree/main/charts);
-   [Docker Swarm](https://github.com/neicnordic/LocalEGA-deploy-swarm/).

Configuration
-------------

Starting the SDA submission services require a running Database and
Message Broker, the setup for those components is detailed in:

- [Database Setup](db.md);
- [Local Message Broker](connection.md#local-message-broker).

[Data Retrieval API](dataout.md) requires a working Database in order to be set up.
