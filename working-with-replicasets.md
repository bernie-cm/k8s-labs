# What are ReplicaSets and how to work with them

ReplicaSets are Kubernetes objects, like everything else in k8s. Their primary function is to ensure that a stable number of replica pods are always running the cluster in accordance with the spec in the YAML file. For this reason, they are instrumental in providing high availability for your application. Typically, you don't work directly with ReplicaSets but rather with Deployments which are a higher level method of ensuring the current state matches the desired state of the cluster.


