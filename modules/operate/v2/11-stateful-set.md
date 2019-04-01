# StatefulSet

## Module Objectives

1. Use StatefulSet to deploy a mysql galera cluster

---

## Converting mysql Deployment to a Stateful Set

When you are using Deployments to manage your Pods it's very easy to scale stateless applications, like our backend and frontend, but scaling stateful applications is more difficult. If you want to scale a mysql database you have to start the first node in a bootstrap mode, then you have to wait until this node is ready and finally you have to start all other nodes and add them to the cluster. If you try to use a Deployment to scale your database, it will create 3 instances of the database Pod simultaneously and you will not be able to create a cluster. StatefulSet is the right object to do such kind of Deployment. Now let's use a StatefulSet to convert our mysql Pod to a 3-node galera mysql cluster.

1. In the `sample-app` folder create a subfolder `mysql-galera`.

1. Inside `mysql-galera` folder create a Dockerfile with the following content.

    ```
    FROM ubuntu:16.04
    ENV DEBIAN_FRONTEND noninteractive

    RUN apt-get update
    RUN apt-get install -y software-properties-common
    RUN apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 BC19DDBA
    RUN add-apt-repository 'deb http://releases.galeracluster.com/mysql-wsrep-5.7/ubuntu xenial main'
    RUN add-apt-repository 'deb http://releases.galeracluster.com/galera-3/ubuntu xenial main'
    RUN apt-get update
    RUN apt-get install -y galera-3 galera-arbitrator-3 mysql-wsrep-5.7 rsync lsof host
    COPY my.cnf /etc/mysql/my.cnf
    COPY start.sh /start.sh
    ENTRYPOINT ["/start.sh"]
    ```

    This Dockerfile simply installs Galera using the Codership repository and copies `my.cnf` and `start.sh` over.

1. Add `my.cnf` file to the `mysql-galera` folder.

    ```
    [mysqld]
    user = mysql
    bind-address = 0.0.0.0
    wsrep_provider = /usr/lib/galera/libgalera_smm.so
    wsrep_sst_method = rsync
    default_storage_engine = innodb
    binlog_format = row
    innodb_autoinc_lock_mode = 2
    innodb_flush_log_at_trx_commit = 0
    query_cache_size = 0
    query_cache_type = 0
    ```

    This file configures galera replication.

1. Add `start.sh` file to the `mysql-galera` folder.

    ```shell
    #!/bin/bash -e

    mkdir /var/run/mysqld
    chown mysql:mysql /var/run/mysqld

    node_list=$(host $SERVICE_NAME | grep "has address" |  awk '{print $4}' | paste -s -d, -)

    if [ -z "$node_list" ]; then
        # if we are in a bootstrap mode we want to set root password first
        mysqld --wsrep-cluster-address=gcomm://$node_list &
        pid="$!"
        # waiting for mysql to become ready
        while !(mysqladmin ping)
        do
           sleep 3
           echo "waiting for mysql ..."
        done
        # setting root password to $MYSQL_ROOT_PASSWORD
        mysql -u root  -e "use mysql; update user set authentication_string=password(\"$MYSQL_ROOT_PASSWORD\") where user='root';"

        # Note: By default, galera-cluster resricts access to localhost only, we are updating this to allow from all IPs
        mysql -u root -e "GRANT ALL ON *.* to root@'%' IDENTIFIED BY 'root';"

        # stopping mysql because we have to restart after setting root password
        if ! kill -s TERM "$pid" || ! wait "$pid"; then
            echo >&2 'MySQL init process failed.'
            exit 1
        fi
    fi
    mysqld --wsrep-cluster-address=gcomm://$node_list
    ```

    `$SERVICE_NAME` is the DNS name of the service that wraps all 3 mysql nodes. This is a headless service, so instead of a virtual IP address it resolves to the list of all IP addresses of the underlying Pods. When the first node is started `host $SERVICE_NAME` command returns and empty list. In this case `node_list` variable is empty. After the first node is started the command returns its IP address, after the second one is started it returns 2 IP addresses separated by comma. This is exactly what we need to start the galera cluster: the first node should be started with `--wsrep-cluster-address=gcomm://` parameter, the second with `--wsrep-cluster-address=gcomm://<first-node-ip>`, the third with `--wsrep-cluster-address=gcomm://<first-node-ip>,<second-node-ip>` and so on.

1. Make `start.sh` executable.

    ```shell
    chmod +x start.sh
    ```

1. Build the image and push it to the container registry.

    ```shell
    docker build . -t gcr.io/$PROJECT_ID/mysql-galera
    docker push gcr.io/$PROJECT_ID/mysql-galera
    ```

1. Delete the `db` Deployment and `db` Service.

    ```shell
    kubectl delete deployment db
    kubectl delete svc db
    ```

1. Save the following file as `manifests/mysql-galera-svc.yaml` and apply the changes.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: galera-cluster
    spec:
      ports:
      - port: 3306
        name: mysql
      clusterIP: None
      selector:
        app: gceme
        role: db
      publishNotReadyAddresses: false
    ```

    The most important parameter here is `clusterIP: None`. This is called a [Headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services). In this case Kubernetes will not create a cluster IP for the service and DNS name `galera-cluster` will point directly to the list of underlying Pods.

1. Save the following file as `manifests/mysql-galera.yaml` and apply the changes.

    ```yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: db
    spec:
      serviceName: "galera"
      replicas: 3
      selector:
        matchLabels:
          app: gceme
          role: db
      template:
        metadata:
          labels:
            app: gceme
            role: db
        spec:
          containers:
          - name: mysql
            image: <REPLACE_WITH_YOUR_OWN_MYSQL_IMAGE>
            ports:
            - containerPort: 3306
              name: mysql
            - containerPort: 4444
              name: sst
            - containerPort: 4567
              name: replication
            - containerPort: 4568
              name: ist
            env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql
                  key: password
            - name: SERVICE_NAME
              value: galera-cluster
            readinessProbe:
              exec:
                command:
                - sh
                - -c
                - "mysql -u root -p$MYSQL_ROOT_PASSWORD -e 'show databases;'"
              initialDelaySeconds: 5
              timeoutSeconds: 2
              successThreshold: 2
    ```

    > Note: Replace the image name with the image just created.

1. Edit the backend deployment `manifests/backend.yaml`.

    1. If you still have the init and multi container remove them, leaving only the backend container, remember to re-add `-run-migrations` to the original backend container.

    1. In the startup command, change `-db-host=db` to `-db-host=galera-cluster`.

    1. Apply the changes.

1. Connect to the app and add some notes.

1. Exec inside one of the db Pods.

    ```shell
    kubectl exec -it db-0 bash
    ```

1. Connect to the mysql database.

    ```shell
    mysql -u root -p$MYSQL_ROOT_PASSWORD
    ```

1. Show databases.

    ```shell
    mysql> show databases;
    ```

    ```
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    ```

1. Show your notes.

    ```sql
    select * from sample_app.notes;
    ```

    ```
    +----+---------------------+---------------------+------------+---------------------+
    | id | created_at          | updated_at          | deleted_at | note                |
    +----+---------------------+---------------------+------------+---------------------+
    |  1 | 2018-11-30 09:09:32 | 2018-11-30 09:09:32 | NULL       | Where are you from? |
    |  2 | 2018-11-30 09:09:48 | 2018-11-30 09:09:48 | NULL       | I am from Ireland   |
    +----+---------------------+---------------------+------------+---------------------+
    2 rows in set (0.00 sec)
    ```

## Optional Exercises

### Use Persistent Volumes in the Stateful Set

Modify `mysql-galera.yaml` to use Persistent Volume Claims. Make sure the data survives after you delete and recreate the database.

