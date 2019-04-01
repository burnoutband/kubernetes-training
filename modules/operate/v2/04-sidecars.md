# Sidecars and Init Containers

## Module Objectives

1. Use sidecars and init containers

---

## Use Sidecars and Init Containers

On startup our `backend` Pod creates a database for itself if it doesn't exist and run migrations. However, usually we want to externalize such tasks from the application Pod. We can use Init Containers to do that.

First, let's verify that the app will fail if we restart the db and don't run migrations.

1. Delete the `-run-migrations` parameters from the `backend` startup command.

1. Delete and recreate the `backend` Pod. At this point the app should work fine because we are still using an old database.

1. Delete and recreate the `db` Pod.

1. Open the app UI you should see a database error such as `Error 1049: Unknown database 'sample_app'`.

    Now let's fix the error by adding an Init Container to the `backend` pod, causing it to run migrations each time before it is started.

1. Add the following section to the `manifests/backend.yaml`.

    ```yaml
    initContainers:
    - name: init-db
      image: <REPLACE_WITH_YOUR_OWN_IMAGE>
      env:
      - name: MYSQL_ROOT_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysql
            key: password
      command: ["sh", "-c", "app -run-migrations -port=8080 -db-host=db -db-password=$MYSQL_ROOT_PASSWORD" ]
    ```

    > Note: You can append these lines directly to the end of the file. The `initContainers` section should have the same number of spaces as the `containers` section under the `spec` section.

1. Recreate the backend Pod.

1. Make sure the app is working fine.


## Optional Exercises

### Use sidecar containers

A Pod can host multiple containers, not just one. Let's try to extend the `backend` Pod and add one more container into it. This Pod can run any image. The startup command should be `sleep 100000`. After the Pod is ready, try to exec into the second container and access the `backend` app using `localhost`.

<details><summary>SOLUTION - CLICK ME</summary>
<p>

Modify the `manifests/backend.yaml` to include an additional container.

```yaml
spec:
  containers:
  - name: multi
    image: busybox
    command: ['sh', '-c', 'echo Multi-container example! && sleep 100000']
```

> Note: in yaml `-` is used for arrays, every object with `-` supports multiple values.

```shell
kubectl exec -i -t backend -c multi /bin/sh
```

</p>
</details>


