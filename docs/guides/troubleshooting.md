# Troubleshooting SDA services

> NOTE:
> Content will continue to be added to this guide as we work on the project.
> If you have feedback to give on the content you would like to see, please contact us on
> [github](https://github.com/neicnordic/neic-sda)!

Exactly what's happened when a container is acting up can be hard to pinpoint.
In this guide we aim to give some general tips on how to troubleshoot and restore SDA services to working order.

## Unresponsive containers

Sometimes containers freeze up.
We do our best to avoid it, but sooner or later it's gonna happen.

The first thing to do in these cases should always be to restart.

Using kubernetes container pods can be restarted using kubectl:
```
kubectl rollout restart <type>/<service>
```
where type is usually `deployment` for SDA pipeline services, or `statefulset` for databases.
ex.
```
kubectl rollout restart deployment/sda-ingest
```

## containers refusing to start

Most of the SDA services are programmed to fail quickly and restart on errors.
This means that misconfigured containers will commonly fail and restart over and over,
trying to connect to other parts of the system.
The logs will commonly tell you which remote container is refusing connection,
and then it's a matter of figuring out the reason.

 - Make sure that the remote container is running and reachable

 - Make sure that the configuration is correct in the configmaps or secrets,
   and that it's correctly mounted into the container.

### Troubleshooting kubernetes

There are some general things that you can do to figure out what problems are in kubernetes:

- `describe` shows the detailed state of kubernetes resources.
  This will often give a clue to what has gone wrong.
  The documentation page on
  [Viewing and finding resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits)
  is a good place to start learning how to find information in kubernetes.

- `top` is only available if there is a metrics server available.
  `top` will show the resources available at the configured kubernetes nodes.
  Kubernetes pods can have
  [resource requests](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/).
  If there are not enough resources available kubernetes will not deploy every service.
  See
  [Requests and limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits)
  for details.

- **service** and **ingress**,
  while a container itself might be running just fine,
  it will be unreachable if the corresponding service isn't running.
  If the container is to be reachable from the outside it also needs an ingress,
  which needs to be configured to talk to the correct service.
  See
  [Services, Load Balancing, and Networking](https://kubernetes.io/docs/concepts/services-networking/) for details on how kubernetes deals with these things.
