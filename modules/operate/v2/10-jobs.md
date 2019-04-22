# Jobs 

## Module Objectives

- Use jobs and cronJobs to schedule task execution

---

## Use Jobs and CronJobs to Schedule Task Execution

Sometimes there is a need to run one-off tasks. You can use Pods to do that, but if the task fails nobody will be able to track that and restart the Pod. Kubernetes Jobs provide a better alternative to run one-off tasks. Jobs can be configured to retry failed task several times. If you need to run a Job on a regular basis you can use CronJobs. Now let's create a CronJob that will do a database backup for us.

1. Save the following file as `manifests/backup.yaml` and apply the changes.

    ```yaml
    apiVersion: batch/v1beta1
    kind: CronJob
    metadata:
      name: backup
    spec:
      schedule: "*/1 * * * *"
      jobTemplate:
        spec:
          backoffLimit: 2
          template:
            spec:
              containers:
              - name: backup
                image: mysql:5.6
                command: ["/bin/sh", "-c"]
                args:
                - mysqldump -h db -u root -p$MYSQL_ROOT_PASSWORD sample_app
                env:
                - name: MYSQL_ROOT_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: mysql
                      key: password
              restartPolicy: OnFailure
    ```

    This creates a CronJob that runs each minute. `backoffLimit` is set to 2 so the job will be retried 2 times in case of failure. The job runs the `mysqldump` command that prints the contents of the `sample_app` database to stdout. This will be available in the Pod logs. (And yes, I know that Pod logs is the worst place to store a database backup 😄  )

1. Get the cronjob status.

    ```yaml
    kubectl get cronjob backup
    ```

    ```
    NAME      SCHEDULE      SUSPEND   ACTIVE    LAST SCHEDULE   AGE
    backup    */1 * * * *   False     0         2m              25m
    ```

1. After 1 minute a` backup` Pod will be created, if you list the Pods you should see the backup completed.

    ```shell
    watch kubectl get pod
    ```

    ```
    NAME                        READY     STATUS      RESTARTS   AGE
    backend-dc656c878-v5fh7     1/1       Running     0          3m
    backup-1543535520-ztrpf     0/1       Completed   0          1m
    db-77df47c5dd-d8zc2         1/1       Running     0          1h
    frontend-654b5ff445-bvf2j   1/1       Running     0          1h
    ```

1. View the `backup` Pod logs and make sure that backup was completed successfully.

    ```shell
    kubectl logs <backup-pod-name>
    ```

1. Delete the CronJob.

    ```shell
    kubectl delete cronjob backup
    ```

## Optional Exercises

###  Use jobs

Use Job instead of CronJob to run the backup only once. 
