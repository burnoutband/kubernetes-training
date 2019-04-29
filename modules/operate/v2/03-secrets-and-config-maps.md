# Secrets and ConfigMaps

## Module Objectives

1. Use Secrets and ConfigMaps to externalize application credentials and configuration

---

## Use Secrets and ConfigMaps to Externalize Application Credentials and Configuration

One major problem with our current deployment is that we hardcoded the MySQL root password in the Pod configuration file. In most cases, we need to externalize secrets and configuration from the Kubernetes object definition. We can use Secrets and ConfigMaps to do that.

1. Create a Secret with the MySQL administrator password.

    ```shell
    kubectl create secret generic mysql --from-literal=password=root
    ```

1. Expose the `mysql` Secret as an environment variable in the `db` Pod. Modify the `env` section in the `manifests/db.yaml` to look like the following.

    ```yaml
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql
          key: password
    ```

    Here we are telling Kubernetes to get the value for the `MYSQL_ROOT_PASSWORD` variable from the `mysql` Secret. Each Secret can have multiple key-value pairs, in our case we get the value from the `password` key.

1. Add exactly the same `env` section to the `manifests/backend.yaml`.

1. Modify the startup command in the `manifests/backend.yaml` file.

    ```yaml
    command: ["sh", "-c", "app -mode=backend -run-migrations -port=8080 -db-host=db -db-password=$MYSQL_ROOT_PASSWORD" ]
    ```

    As you can see, here we call `sh` instead of calling our app directly. Shell is required to do environment variable substitution for us. Also we use `$MYSQL_ROOT_PASSWORD` instead of hardcoded password.

1. Redeploy `backend` and `db`. The database Pod should be redeployed first, because the backend creates an empty database on startup and this database will be destroyed if you redeploy the database after the backend.

    > Note: You will not be able to use `kubectl apply` command this time. Instead, you should use `kubectl delete` first and then redeploy pod.

1. Refresh the frontend and make sure that the app is still working fine.

## Optional Exercises

### ConfigMaps

1. Create a config map with a key that contains port for the backetd. Use the value from the config map in the backend startup command.
