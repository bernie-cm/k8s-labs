# What are ReplicaSets and how to work with them

iThey are objects in k8s that ensure a stable number of replica pods are always running. ReplicaSets assist with high availability (HA) in k8s because they ensure that a specified number of replica Pods are always running. For example, if the ReplicaSet specifies that 3 replicas are needed, and only 2 are running then it will run another one.
