# CronJob with concurrencyPolicy: Allow

Note:

> It specifies how to treat concurrent executions of a job that is created by this
> cron job.
>
> Allow (default): The cron job allows concurrently running jobs

This time we increase the sleep to 90 seconds

The concurrencyPolicy was irrelevant in its case: no two jobs / Pods were ever
able to run simultaneously.

This example will demonstrate concurrencyPolicy in action.

Every minute a new job must run.

Our Pod must sleep for 90 seconds.

So after 60 seconds - when Pod still has 30 seconds of sleeping work left - another
Pod will be automatically scheduled.

concurrencyPolicy: Allow will allow this existing Pod to continue running for the
full 90 seconds.

The policy will also allow the simultaneous starting and running of the second
newly scheduled Pod.

```bash
kubectl create -f mycronjob.yaml

cronjob.batch/mycronjob created
```

Run

```bash
kubectl get cronjob
NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mycronjob   */1 * * * *   False     0        <none>          15s
```

Many seconds later, first Pod starts:

```bash
kubectl get job
mycronjob-1548665220   0/1           4s         4s

kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
mycronjob-1548665220-8pdnw   1/1     Running   0          4s
```

Important ... see below: One minute later, new Pod starts up, and old one continues
to run.

Note the old job is still running, its COMPLETIONS is still zero.

The old pod is also READY ... it is still running.

```bash
kubectl get job
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548665220   0/1           63s        63s
mycronjob-1548665280   0/1           3s         3s

kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
mycronjob-1548665220-8pdnw   1/1     Running   0          64s
mycronjob-1548665280-6cnjm   1/1     Running   0          4s
```

So now we have two jobs, each with one pod running in parallel.

```bash
kubectl get job
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548665220   0/1           76s        76s
mycronjob-1548665280   0/1           16s        16s


kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
mycronjob-1548665220-8pdnw   1/1     Running   0          76s
mycronjob-1548665280-6cnjm   1/1     Running   0          16s
```

After 90 seconds total runtime.

```bash
kubectl get job
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548665220   1/1           92s        94s
mycronjob-1548665280   0/1           34s        34s

kubectl get pods
NAME                         READY   STATUS      RESTARTS   AGE
mycronjob-1548665220-8pdnw   0/1     Completed   0          94s
mycronjob-1548665280-6cnjm   1/1     Running     0          34s
```

Above: first job is complete, second job is running.

Below: after several minutes: one job started every minute.

First 3 jobs show COMPLETIONS equal to 1 ( success ). DURATION correct for completed jobs.

```bash
kubectl get job
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548665280   1/1           92s        4m2s
mycronjob-1548665340   1/1           91s        3m1s
mycronjob-1548665400   1/1           92s        2m1s
mycronjob-1548665460   0/1           61s        61s
mycronjob-1548665520   0/1           1s         1s
```

History of completed Pods.

```bash
kubectl get pods
NAME                         READY   STATUS              RESTARTS   AGE
mycronjob-1548665280-6cnjm   0/1     Completed           0          4m2s
mycronjob-1548665340-wm2zk   0/1     Completed           0          3m1s
mycronjob-1548665400-nnfgk   0/1     Completed           0          2m1s
mycronjob-1548665460-crn6x   1/1     Running             0          61s
mycronjob-1548665520-qt7zd   0/1     ContainerCreating   0          1s
```

3 completed pods are no READY anymore. ( Completed successfully )

One pod has been running 61 seconds and is still ready. It still has 29 seconds
of sleeping to do.

Our cron job precise minute timer kicked off another Pod that is still in
preliminary ContainerCreating state.

By default 3 successfully done jobs are kept.

Minutes later ... we can see 2 old jobs deleted (near bottom)

```bash
kubectl describe cronjob/mycronjob

Events:
  Type    Reason            Age    From                Message
  ----    ------            ----   ----                -------
  Normal  SuccessfulCreate  5m44s  cronjob-controller  Created job mycronjob-1548665220
  Normal  SuccessfulCreate  4m44s  cronjob-controller  Created job mycronjob-1548665280
  Normal  SawCompletedJob   4m3s   cronjob-controller  Saw completed job: mycronjob-1548665220
  Normal  SuccessfulCreate  3m43s  cronjob-controller  Created job mycronjob-1548665340
  Normal  SawCompletedJob   3m3s   cronjob-controller  Saw completed job: mycronjob-1548665280
  Normal  SuccessfulCreate  2m43s  cronjob-controller  Created job mycronjob-1548665400
  Normal  SawCompletedJob   2m3s   cronjob-controller  Saw completed job: mycronjob-1548665340
  Normal  SuccessfulCreate  103s   cronjob-controller  Created job mycronjob-1548665460
  Normal  SawCompletedJob   63s    cronjob-controller  Saw completed job: mycronjob-1548665400
  
  Normal  SuccessfulDelete  63s    cronjob-controller  Deleted job mycronjob-1548665220 ..........
  
  Normal  SuccessfulCreate  43s    cronjob-controller  Created job mycronjob-1548665520
  Normal  SawCompletedJob   3s     cronjob-controller  Saw completed job: mycronjob-1548665460
  
  Normal  SuccessfulDelete  3s     cronjob-controller  Deleted job mycronjob-1548665280 ...........
```

Demo done

```bash
kubectl delete -f mycronjob.yaml 

cronjob.batch "mycronjob" deleted
```
