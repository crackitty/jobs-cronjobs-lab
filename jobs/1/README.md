# A Job using one Pod that runs once

```bash
kubectl apply -f myjob.yaml

> job.batch/myjob created
```

Let's list all Pods by running kubectl get pods several times.

```bash
kubectl get pods

NAME          READY   STATUS    RESTARTS   AGE
myjob-hbhtl   1/1     Running   0          2s

NAME          READY   STATUS    RESTARTS   AGE
myjob-hbhtl   1/1     Running   0          4s

NAME          READY   STATUS    RESTARTS   AGE
myjob-hbhtl   1/1     Running   0          6s

NAME          READY   STATUS    RESTARTS   AGE
myjob-hbhtl   1/1     Running   0          9s

NAME          READY   STATUS      RESTARTS   AGE
myjob-hbhtl   0/1     Completed   0          12s

NAME          READY   STATUS      RESTARTS   AGE
myjob-hbhtl   0/1     Completed   0          16s
```

We see the Pod ran for around 10 seconds. Then the status becomes Completed.

```bash
kubectl delete -f myJob.yaml

job "myjob" deleted
```

Here we used `kubectl get pods` to monitor the progress of our job

We can also monitor the job with `kubectl get job`

```bash
kubectl get job

NAME    COMPLETIONS   DURATION   AGE
myjob   0/1           3s         3s

NAME    COMPLETIONS   DURATION   AGE
myjob   0/1           5s         5s

NAME    COMPLETIONS   DURATION   AGE
myjob   0/1           7s         7s

NAME    COMPLETIONS   DURATION   AGE
myjob   0/1           9s         9s

NAME    COMPLETIONS   DURATION   AGE
myjob   1/1           12s        12s
```

See COMPLETIONS 1/1 ? Done! clean up!

```bash
kubectl delete -f myjob.yaml

> job.batch "myjob" deleted
```

Next - [an example with erroring container and backoffLimit](../2/README.md)
