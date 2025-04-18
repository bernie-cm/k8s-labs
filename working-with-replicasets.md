# What are ReplicaSets and how to work with them

ReplicaSets are Kubernetes objects, like everything else in k8s. Their primary function is to ensure that a stable number of replica pods are always running the cluster in accordance with the spec in the YAML file. For this reason, they are instrumental in providing high availability for your application. Typically, you don't work directly with ReplicaSets but rather with Deployments which are a higher level method of ensuring the current state matches the desired state of the cluster.

## Obtaining information
The most basic method to list the number of ReplicaSets is using `kubectl`.
```bash
$ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       5m12s
```
Here we can see that there are 4 desired pods in the RepicaSet spec, and there are 4 pods running. Even if we deleted one of the pods with `kubectl delete pod <name-of-pod>`, the ReplicaSet will ensure another pod is running to match the number of replicas, i.e., 4.

## Creating a ReplicaSet
Like most objects in k8s, the easiest way to create a ReplicaSet is using a YAML file.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-set-1
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
      - image: nginx
```

Once the YAML has been defined, it's very simple to create the ReplicaSet.
```bash
$ kubectl apply -f replica-set.yaml`
replicaset.apps/replicaset-1 created
```
