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
 
   When deploying applications on Kubernetes, it is essential to understand the DNS naming conventions and ingress configurations for Pods and Services. Each Pod within the cluster is assigned a DNS name in the format of `pod-ip-address.<cluster>.pod.cluster.local`. This DNS resolution allows seamless communication between Pods within the same cluster.

   Services, representing sets of Pods, are assigned A DNS records with names structured as `<service_name>.<namespace>.svc.cluster.local`. This DNS record resolves to the cluster IP of the respective Service.

    | Service Name | Common DNS Name                         |
    | ------------ | ----------------------------------------|
    | inbox        | sda-svc-inbox.<namespace>.svc.cluster.local   |
    | download     | sda-svc-download.<namespace>.svc.cluster.local|
    | auth         | sda-svc-auth.<namespace>.svc.cluster.local    |
    | mq           | broker-sda-mq.<namespace>.svc.cluster.local   |

    Certain services, such as `inbox`, `download`, and `auth`, are configured to expect an ingress. Ingress provides external access to these services, allowing external clients to communicate with them. The following services specifically expect an ingress:

    - inbox
    - download
    - auth

    In addition, Kubernetes allows you to define Network Policies to control the communication between Pods. Network Policies are crucial for enforcing security measures within your cluster. They enable you to specify which Pods can communicate with each other and define rules for ingress and egress traffic.
    Here's a basic example of a Network Policy that allows traffic only to the 'inbox', 'download' and 'auth' service:

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
    name: allow-auth-service
    spec:
    podSelector: {}
    ingress:
    - from:
        - podSelector:
            matchLabels:
            app: sda-svc-inbox
        - podSelector:
            matchLabels:
            app: sda-svc-download
        - podSelector:
            matchLabels:
            app: sda-svc-auth
    ```

## Complementary services

 - sda-auth, sda-doa, sda-download


