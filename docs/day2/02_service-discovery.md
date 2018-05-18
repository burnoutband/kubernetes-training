## Service Discovery

Service discovery is the process of determining how to connect to a service.
While there is a service discovery option based on [environment
variables](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/#environment-variables)
available, the DNS-based service discovery is preferable.

>Note that DNS is a [cluster add-on](https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/dns/README.md)
so be sure your Kubernetes distribution provides one or install it
yourself.

### Exercise 1: Deploying services and testing DNS
