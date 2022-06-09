# CronJob lab

```bash
kubectl create cronjob onetimejob --image=crackitty/onetimejob:1.0 --schedule="*/1 * * * *" --dry-run=client -o yaml > onetimejob.yaml
```

Dry-run creates the yaml and writes it to `onetimejob.yaml`

```bash
kubectl apply -f onetimejob.yaml
```

This schedules the onetimejob to be run every minute. So it will be created, executed
and then it will be destroyed when it finishes execution.

The default setup means that we will be running these tasks one after another, but
what happens if the job runs past the scheduled time for a new job to start?

The default is to allow the new job to be scheduled in parallel but you can configure
that.

## Labs

- [Jobs](./jobs/README.md)
- [CronJobs](./cronjobs/README.md)
