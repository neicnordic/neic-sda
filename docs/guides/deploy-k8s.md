# Deploying on Kubernetes

> TODO:
> This guide is a stub and has yet to be finished.
> If you have feedback to give on the content you would like to see, please contact us on
> [github](https://github.com/neicnordic/neic-sda)!


## Guide summary

This guide explains how to deploy the Sensitive Data Archive (SDA) in kubernetes.
- What it intends to cover
- What to expect, scope, explain level of details
- How self-contained the guide is
- Examples expected to work directly or not, must be configured (example configurations, most updated version?)

## Local security / zone considerations

- Differences in deployment make concrete examples challenges, explain what can be exemplified and what not in this guide


For secure deployment of the system one can think it by what can be accessed from where, for all ways of deploying two trust boundaries can be used, external and internal. For an extra layer of security also the storage trust boundary can be separate. The service is provided for customers on the internet therefore an example of deploying the service is using two separate Kubernetes clusters, one for responding customers and other communication outside, and the other cluster is more secure storage facing internal cluster. One thing to consider is where to release the data, that could be closed protected environment with tightly restricted access, Data out can be put in internal cluster.

The services could be divided into two trust boundaries
- The services in external in external cluster are Inbox and MQ
- The services in internal cluster are  Intercept, Ingest, Verify, Mapper, Finalize, Backup and Data out.

The innermost trust zone contains the database and the archive, which be accessed only from internal cluster.



## Charts overview

## System requirements

 - k8s/helm minimal versions
 - rough estimate of hardware resources

## Minimal working configuration

## Security issues

 - Enabling TLS example
 - Secret handling example

## Network policies

 - DNS names and ingress for services

## Complementary services

 - sda-auth, sda-doa, sda-download


