# A Parallel Job with a fixed completion count - with parallelism at 2

So now we've upped the parallelism to 2

```bash
kubectl apply -f myjob.yaml

> job/myjob created
```

Let's see the progress

```bash
kubectl get pods

NAME          READY   STATUS              RESTARTS   AGE
myjob-11111   0/1     ContainerCreating   0          2s
myjob-22222   0/1     ContainerCreating   0          2s

NAME          READY   STATUS    RESTARTS   AGE
myjob-11111   1/1     Running   0          4s
myjob-22222   1/1     Running   0          4s

NAME          READY   STATUS      RESTARTS   AGE
myjob-33333   1/1     Running     0          1s
myjob-44444   1/1     Running     0          1s
myjob-11111   0/1     Completed   0          7s
myjob-22222   0/1     Completed   0          7s

NAME          READY   STATUS      RESTARTS   AGE
myjob-33333   1/1     Running     0          3s
myjob-44444   1/1     Running     0          3s
myjob-11111   0/1     Completed   0          9s
myjob-22222   0/1     Completed   0          9s

NAME          READY   STATUS      RESTARTS   AGE
myjob-33333   0/1     Completed   0          5s
myjob-44444   0/1     Completed   0          5s
myjob-11111   0/1     Completed   0          11s
myjob-22222   0/1     Completed   0          11s
```

We set `completions: 4` Therefore 2 Pods were run in parallel twice to get
the completions done.

We can do the same using `kubectl get jobs myjob`

We see something like this

```bash
kubectl get jobs myjob

NAME    COMPLETIONS   DURATION   AGE
myjob   0/4           2s         2s

NAME    COMPLETIONS   DURATION   AGE
myjob   0/4           5s         5s

NAME    COMPLETIONS   DURATION   AGE
myjob   2/4           9s         9s

NAME    COMPLETIONS   DURATION   AGE
myjob   4/4           9s         11s
```

We can see them running in batches of 2 - in parallel

```bash
kubectl delete -f myjob.yaml

job "myjob" deleted
```

## Extra - activeDeadlineSeconds

> Once a Job reaches activeDeadlineSeconds, all of its Pods are terminated and
> the Job status will become type: Failed with reason: DeadlineExceeded.

So if we uncomment the last 2 lines in our yaml and rerun everything, then we'll
see that we are now running 4 pods in parallel but with an `activeDeadlineSecons`
at 3 seconds so after 3 seconds they all get terminated - as per the spec.

```bash
kubectl apply -f myjob-deadline.yaml

> job/myjob created
```

The pods will disappear quickly so better to look at the job

```bash
kubectl describe job/myjob
Name:                     myjob
Namespace:                default
Parallelism:              4
Completions:              4
Start Time:               Thu, 24 Jan 2019 08:25:53 +0200
Active Deadline Seconds:  3s
Pods Statuses:            0 Running / 0 Succeeded / 4 Failed
Events:
  Type     Reason            Age   From            Message
  ----     ------            ----  ----            -------
  Normal   SuccessfulCreate  8s    job-controller  Created pod: myjob-5rcwp
  Normal   SuccessfulCreate  8s    job-controller  Created pod: myjob-nbdz4
  Normal   SuccessfulCreate  8s    job-controller  Created pod: myjob-4jb28
  Normal   SuccessfulCreate  7s    job-controller  Created pod: myjob-vb765
  
  Normal   SuccessfulDelete  4s    job-controller  Deleted pod: myjob-vb765
  Normal   SuccessfulDelete  4s    job-controller  Deleted pod: myjob-4jb28
  Normal   SuccessfulDelete  4s    job-controller  Deleted pod: myjob-5rcwp
  Normal   SuccessfulDelete  4s    job-controller  Deleted pod: myjob-nbdz4
  
  Warning  DeadlineExceeded  4s    job-controller  Job was active longer than specified deadline
```

Much more helpful: 0 Succeeded / 4 Failed

Also informative: Warning - - DeadlineExceeded - - Job was active longer than
specified deadline

Now we see why our Pods disappeared ... they were deleted.

I do not understand the logic:

when Pods complete successfully they continue to exist so you can inspect the
SUCCESS job logs when Pods complete unsuccessfully they get deleted so you CANNOT
inspect the FAILED job logs.

One way around this is to add the following:

```yaml
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5
```

Which tells K8s to hold onto the pods for the last 5 failed and 3 successful jobs