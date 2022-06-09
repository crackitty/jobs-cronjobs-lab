# A Job using one Pod that runs once - but fails!

```bash
kubectl apply -f myjob.yaml

> job.batch/myjob created
```

Let's list all Pods by running kubectl get pods several times.

```bash
kubectl get pods

NAME          READY   STATUS              RESTARTS   AGE
myjob-drzhx   0/1     ContainerCreating   0          2s
myjob-hwdc5   0/1     Error               0          3s

NAME          READY   STATUS   RESTARTS   AGE
myjob-drzhx   0/1     Error    0          5s
myjob-hwdc5   0/1     Error    0          6s

NAME          READY   STATUS   RESTARTS   AGE
myjob-drzhx   0/1     Error    0          9s
myjob-hwdc5   0/1     Error    0          10s

NAME          READY   STATUS   RESTARTS   AGE
myjob-drzhx   0/1     Error    0          14s
myjob-gks4f   0/1     Error    0          4s
myjob-hwdc5   0/1     Error    0          15s
```

With `backoffLimit: 2` the job stops creating containers after the third error

Let's describe the job and see what happened

```bash
kubectl describe job/myjob

Name:           myjob
Parallelism:    1
Completions:    1
Start Time:     Thu, 24 Jan 2019 07:50:07 +0200
Pods Statuses:  0 Running / 0 Succeeded / 3 Failed
Pod Template:
  Containers:
   myjob:
    Command:
      sh
      -c
      echo Job Pod is Running ; exit 1
  
Events:
  Type     Reason                Age   From            Message
  ----     ------                ----  ----            -------
  Normal   SuccessfulCreate      35s   job-controller  Created pod: myjob-hwdc5
  Normal   SuccessfulCreate      34s   job-controller  Created pod: myjob-drzhx
  Normal   SuccessfulCreate      24s   job-controller  Created pod: myjob-gks4f
  Warning  BackoffLimitExceeded  4s    job-controller  Job has reached the specified backoff limit
```

Clean up

```bash
kubectl delete -f myjob.yaml

> job.batch "myjob" deleted
```
