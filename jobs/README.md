# Jobs demos

We're going to demo the following type of `Jobs`

1. [A Job using one Pod that runs once](./1/README.md)
   1. [with failing workload and backoffLimit](./2/README.md)
2. [A Parallel Job with a fixed completion count](./3/README.md)
   1. [with a parallelism at 2](./4/README.md)

Note:

Another more advanced pattern is to run Parallel Jobs with a work queue.

Here you would be running some message queue and the Job is
configured to connect to the queue and pull off items and perform
some work on that item. You configure the parallelism and it runs
until the queue is empty.
