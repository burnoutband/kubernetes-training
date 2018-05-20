## Labels and Selectors

Labels are the mechanism you use to organize Kubernetes objects. 

A label is a key-value pair that is meaningful and relevant to users with certain [restrictions](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set) concerning length and allowed values.

### Exercise 1: Labels in pods

1. Save the following as `label-pod.yaml`

    ```console
    apiVersion: v1
    kind: Pod
    metadata:
      name: label-pod
      labels:
        env: development
    spec:
      containers:
      - name: sise
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
    ```

1. Run the following command to create the pod

    ```console
    kubectl create -f label-pod.yaml
    ```

1. Run the following to list the pod, noting the new column

    ```console
    kubectl get pods --show-labels

    NAME       READY     STATUS    RESTARTS   AGE    LABELS
    label-pod  1/1       Running   0          10m    env=development
    ```

1. Add an additional label to the pod by running

    ```console
    kubectl label pods label-pod owner=you

    kubectl get pods --show-labels

    NAME        READY     STATUS    RESTARTS   AGE    LABELS
    label-pod   1/1       Running   0          16m    env=development,owner=you
    ```

### Exercise 2: Using Selectors

We can use labels for filtering a list.

1. To list all pods labeled as `owner=you`, we can use the `--selector` option:

    ```console
    kubectl get pods --selector owner=you

    NAME       READY     STATUS    RESTARTS   AGE
    label-pod  1/1       Running   0          27m
    ```

    >Note: The `--selector` option can be abbreviated to `-l`

1. To list pods labeled with `env=development` we can run:

    ```console
    kubectl get pods -l env=development

    NAME       READY     STATUS    RESTARTS   AGE
    label-pod  1/1       Running   0          27m
    ```

1. Save the following as `label-pod2.yaml`

    ```console
    apiVersion: v1
    kind: Pod
    metadata:
      name: label-pod2
      labels:
        env: production
        owner: you
    spec:
      containers:
      - name: sise
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
    ```

1. Create the above pod using

    ```console
    kubectl create -f label-pod2.yaml
    ```

1. To list all pods labeled with `env=development` or `env=production`

    ```console
    kubectl get pods -l 'env in (production, development)'

    NAME           READY     STATUS    RESTARTS   AGE
    label-pod      1/1       Running   0          43m
    label-pod2     1/1       Running   0          3m
    ```

1. Other verbs also support label selection, for example, you could remove both of these pods with:

    ```console
    kubectl delete pods -l 'env in (production, development)'
    ```

---

Note: Labels are not restricted to pods. In fact, you can apply them to all sorts of objects, such as `nodes` or `services`.

---

### Exercise 3 (Optional): Using Selectors

1. Deploy 3 pods; each one with different labels: `version=1`, `version=2` and `version=3`

1. List the pods using selectors that will return all pods with versions not equal to 3

Refer to the [documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) to review selector syntax.

