# A Parallel Job with a fixed completion count

```bash
kubectl apply -f myjob.yaml

> job/myjob created
```

Let's see the progress

```bash
kubectl get jobs myjob

NAME    COMPLETIONS   DURATION   AGE
myjob   0/4           2s         2s

NAME    COMPLETIONS   DURATION   AGE
myjob   0/4           4s         4s

NAME    COMPLETIONS   DURATION   AGE
myjob   1/4           6s         6s

NAME    COMPLETIONS   DURATION   AGE
myjob   1/4           9s         9s

NAME    COMPLETIONS   DURATION   AGE
myjob   2/4           12s        12s

NAME    COMPLETIONS   DURATION   AGE
myjob   3/4           14s        14s

NAME    COMPLETIONS   DURATION   AGE
myjob   3/4           16s        16s

NAME    COMPLETIONS   DURATION   AGE
myjob   4/4           18s        19s
```

We can clearly see the 4 Pods each sleeping 3 seconds successfully and one-by-one,
not all simultaneously

We can also check the status (as before) using `kubectl get po` to see things
a little differently.

You'll notice that only 1 Pod runs at a time.

> Default parallelism is one

```bash
kubectl delete -f myjob.yaml

job "myjob" deleted
```
