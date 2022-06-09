# CronJobs demos

We're going to demo the following type of `CronJobs`

1. Basic CronJob that runs once a minute
2. CronJob using `concurrencyPolicy: Allow`
3. CronJob using `concurrencyPolicy: Forbid`

Note about `concurrencyPolicy: Replace`

Same as the others except that:

> **Replace** : If it is time for a new job run and the previous job run
> hasn't finished yet, the cron job replaces the currently running job
> run with a new job run
