# Basic CronJob

```bash
kubectl create -f mycronjob.yaml

cronjob.batch/mycronjob created
```

Unfortunately Linux cron job functionality allows per minute as smallest time
increments.

So throughout this tutorial we have to repeatedly wait around a minute before
something happens.

```bash
kubectl get cronjob

NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mycronjob   */1 * * * *   False     0        <none>          14s


kubectl get job

No resources found.
```

After about a minute

```bash
kubectl get job
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548664260   0/1           4s         4s


kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
mycronjob-1548664260-7dhlm   1/1     Running   0          5s
```

Note jobs get prefixed with our cron job name (from our YAML)

A few seconds later

```bash
kubectl get job

NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548664260   1/1           6s         6s


kubectl get po

NAME                         READY   STATUS      RESTARTS   AGE
mycronjob-1548664260-7dhlm   0/1     Completed   0          7s
```

So...

- job has completed: COMPLETIONS 1
- job took 6s ... DURATION
- Pod is Completed
- Pod is no longer available ... READY is 0

Get some more info

```bash
kubectl get cronjob

NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mycronjob   */1 * * * *   False     0        54s             87s
```

So, nearly a minute later ... all that happened is AGE increased ... awaiting
the next precise minute to start another run of this cron job.

30 seconds later

```bash
kubectl get cronjob
NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mycronjob   */1 * * * *   False     1        1s              94s
```

Active is ONE. A Pod for this cron job is running.

```bash
kubectl get job

NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548664260   1/1           6s         63s
mycronjob-1548664320   0/1           3s         3s
```

If we check the pods we see the pod for the new job is running and the old one
is COMPLETED and the AGE is greater.

```bash
kubectl get po
NAME                         READY   STATUS      RESTARTS   AGE
mycronjob-1548664260-7dhlm   0/1     Completed   0          63s
mycronjob-1548664320-75n9z   1/1     Running     0          3s
```

We can, as with the Jobs, describe the resource

```bash
kubectl describe cronjob/mycronjob

Name:                       mycronjob
Schedule:                   */1 * * * *
Concurrency Policy:         Allow
Pod Template:
  Containers:
   mycron-container:
    Command:
      echo Job Pod is Running ; sleep 5
Last Schedule Time:  Mon, 28 Jan 2019 10:33:00 +0200
Active Jobs:         mycronjob-1548664380
Events:
  Type    Reason            Age   From                Message
  ----    ------            ----  ----                -------
  Normal  SuccessfulCreate  2m6s  cronjob-controller  Created job mycronjob-1548664260
  Normal  SawCompletedJob   116s  cronjob-controller  Saw completed job: mycronjob-1548664260
  Normal  SuccessfulCreate  66s   cronjob-controller  Created job mycronjob-1548664320
  Normal  SawCompletedJob   56s   cronjob-controller  Saw completed job: mycronjob-1548664320
  Normal  SuccessfulCreate  6s    cronjob-controller  Created job mycronjob-1548664380
```

It would have been great if this command also showed:

- number of jobs successfully completed
- number of jobs unsuccessfully completed

```bash
kubectl delete -f mycronjob.yaml 
   
cronjob.batch "mycronjob" deleted
```
