# Distributed paperless-ngx setup

The idea is to show how [paperless-ngx](https://github.com/paperless-ngx/paperless-ngx)
can be deployed in a horizontally scalable way.

Currently the container image is a one-man show, and all services (django,  consumer,
celery, celery-beat, celery-flower) all run in one container, which doesn't
follow the one-container/one-process approach.

Since the services are started using supervisord, it's quite easy to split
them into one pod per process, as you can just override the `supervisord.conf`
so that it only starts one process.

## Kubernetes example
The `kubernetes` directory contains all resources required to start this setup,
on any cluster.

The PVCs created in `02_pvc.yml` might need a `storageClass` configured if your
custer doesn't have a default storage class set.

Each deployment, mounts the same configMap as environment variables for
the paperless-ngx configuration (eg. redis, database, hostnames, directory
paths, etc), and the same secret to ensure they are all configured the same way.

Additionally each pod mounts a configMap which overrides the `supervisord.conf`,
but they are all managed in the same configMap.

Last but not least there is a backup-job, which creates a daily backup of the
paperless instance, with all files and configs, to be imported into a separate instance, or to be replicated off-site for an off-site backup.

Ideally one would configure an HorizontalPodAutoscaler to scale the `celery` pods if multiple documents need to be parsed. Since the HPA is limited to resource usage, it might be tricky to configure, and an approach using KEDA might be better. (Check https://github.com/abhirockzz/redis-celery-kubernetes-keda for an example how Keda could be integrated with celery backed by redis.)

But we could also just increase the number of replicas for the celery deployment, that would also allow processing more documents at once.

## docker-compose example
If I get around to it, I'll create a docker-compose setup, mirroring the setup,
I've already done for kuberentes, with one container per process, but without the scaling implemented.