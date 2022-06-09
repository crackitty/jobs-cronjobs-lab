# CronJob with concurrencyPolicy: Forbid

Create the CronJob

```bash
kubectl create -f mycronjob.yaml

cronjob.batch/mycronjob created
```

After a minute:

```bash
kubectl get job
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548666360   0/1           57s        57s

kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
mycronjob-1548666360-gg26f   1/1     Running   0          57s
```

Below: one minute elapsed, original first job still running.

New job forbidden to start. ( We do not see a new job with age of around 18 seconds )

```bash
kubectl get job
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548666360   0/1           78s        78s

NAME                         READY   STATUS    RESTARTS   AGE
mycronjob-1548666360-gg26f   1/1     Running   0          78s
```

Nearly 90 seconds later - first job busy with last seconds of running.

```bash
kubectl get job
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548666360   1/1           92s        95s

kubectl get pods
NAME                         READY   STATUS      RESTARTS   AGE
mycronjob-1548666360-gg26f   0/1     Completed   0          95s
```

First job complete, NOW second job starts immediately.

```bash
NAME                   COMPLETIONS   DURATION   AGE
mycronjob-1548666360   1/1           92s        103s
mycronjob-1548666420   0/1           3s         3s

kubectl get pods
NAME                         READY   STATUS      RESTARTS   AGE
mycronjob-1548666360-gg26f   0/1     Completed   0          103s
mycronjob-1548666420-t92nc   1/1     Running     0          3s
```

And it goes on like this...

kubectl describe output :

```bash
kubectl describe cronjob/mycronjob

Events:
  Type    Reason            Age    From                Message
  ----    ------            ----   ----                -------
  Normal  SuccessfulCreate  3m34s  cronjob-controller  Created job mycronjob-1548666360
  
  Normal  SawCompletedJob   114s   cronjob-controller  Saw completed job: mycronjob-1548666360
  Normal  SuccessfulCreate  114s   cronjob-controller  Created job mycronjob-1548666420
  
  Normal  SawCompletedJob   13s    cronjob-controller  Saw completed job: mycronjob-1548666420
  Normal  SuccessfulCreate  13s    cronjob-controller  Created job mycronjob-1548666540
  ```

  Note the original neat schedule of one new job per minute is now destroyed.

Every new job only start when previous one completes. Not on the minute, but 30
seconds later since each job runs 90 seconds.

Use concurrencyPolicy: Forbid when you want each cron job to fully complete ALL BY
ITSELF, then start the next job in the schedule queue. Subsequent cron job start
times depend on runtime of previous job, not on precise schedule.

kubectl get cronjob does not show any detail of all this that just happened.

```bash
kubectl get cronjob
NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
mycronjob   */1 * * * *   False     1        65s             12m
```

Clean up

```bash
kubectl delete -f mycronjob.yaml 
   
cronjob.batch "mycronjob" deleted
```

