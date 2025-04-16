# Labs for how to manage Kubernetes cluster

## Examining pods
Using `kubectl` we can examine the pods running in the cluster. The simplest way is to run this command.
```bash
# Simple command to examine the pods
$ kubectl get po

# To get information about the pods running in a specific namespace and showing their IP addresses
# in this example the kube-system namespace
$ kubectl get po -o wide -A -n kube-system
```
