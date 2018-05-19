## Stateful Sets

StatefulSet is the workload API object used to manage stateful applications.

Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

### Exercise 1: Deploying Cassandra with Stateful Sets

1. Create a Cassandra Headless Service

    Save the following file as `cassandra-service.yaml`
    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: cassandra
      name: cassandra
    spec:
      clusterIP: None
      ports:
      - port: 9042
      selector:
        app: cassandra
    ```
    This Service is used for DNS lookups between Cassandra Pods and clients within the Kubernetes Cluster.

1. Use a StatefulSet to Create a Cassandra Ring
    Save the following file as `cassandra-statefulset.yaml`
    ```
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: cassandra
      labels:
        app: cassandra
    spec:
      serviceName: cassandra
      replicas: 3
      selector:
        matchLabels:
          app: cassandra
      template:
        metadata:
          labels:
            app: cassandra
        spec:
          terminationGracePeriodSeconds: 1800
          containers:
          - name: cassandra
            image: gcr.io/google-samples/cassandra:v13
            imagePullPolicy: Always
            ports:
            - containerPort: 7000
              name: intra-node
            - containerPort: 7001
              name: tls-intra-node
            - containerPort: 7199
              name: jmx
            - containerPort: 9042
              name: cql
            resources:
              limits:
                cpu: "500m"
                memory: 1Gi
              requests:
               cpu: "500m"
               memory: 1Gi
            securityContext:
              capabilities:
                add:
                  - IPC_LOCK
            lifecycle:
              preStop:
                exec:
                  command: 
                  - /bin/sh
                  - -c
                  - nodetool drain
            env:
              - name: MAX_HEAP_SIZE
                value: 512M
              - name: HEAP_NEWSIZE
                value: 100M
              - name: CASSANDRA_SEEDS
                value: "cassandra-0.cassandra"
              - name: CASSANDRA_CLUSTER_NAME
                value: "K8Demo"
              - name: CASSANDRA_DC
                value: "DC1-K8Demo"
              - name: CASSANDRA_RACK
                value: "Rack1-K8Demo"
              - name: POD_IP
                valueFrom:
                  fieldRef:
                    fieldPath: status.podIP
            readinessProbe:
              exec:
                command:
                - /bin/bash
                - -c
                - /ready-probe.sh
              initialDelaySeconds: 15
              timeoutSeconds: 5
            volumeMounts:
            - name: cassandra-data
              mountPath: /cassandra_data
      volumeClaimTemplates:
      - metadata:
          name: cassandra-data
        spec:
          accessModes: [ "ReadWriteOnce" ]
          resources:
            requests:
              storage: 1Gi
    ```
    Pay attension to CASSANDRA_SEEDS environment variable - nodes use it to discover each other. Make sure you understand how this variable is related to casandra service.

1. Deploy casandra statefull set 
    ```
    kubectl create -f cassandra-statefulset.yaml
    ```

1. Get the Pods to see the ordered creation status:
    ```
    kubectl get pods -l=“app=cassandra”
    ```
    The response should be
    ```
       NAME          READY     STATUS              RESTARTS   AGE
       cassandra-0   1/1       Running             0          1m
       cassandra-1   0/1       ContainerCreating   0          8s
    ```

1. Validate The Cassandra StatefulSet
    ```
    kubectl get statefulset cassandra
    ```

1. Run the Cassandra utility nodetool to display the status of the ring.
    ```
    kubectl exec cassandra-0 – nodetool status
    ```

