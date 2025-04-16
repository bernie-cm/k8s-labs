# Labs for how to manage Kubernetes cluster

## Examining pods
Using `kubectl` we can examine the pods running in the cluster. The simplest way is to run this command.
```bash
# Simple command to examine the pods
$ kubectl get po

# To get information about the pods running in a specific namespace and showing their IP addresses
# in this example the kube-system namespace
$ kubectl get po -n kube-system -o wide
```

## Performing a backup of etcd
The datastore etcd, also known as the cluster store, has all the stateful configuration information about our cluster. One common operation is to back up the etcd datastore using a CLI utility called `etcdctl`.
### Pre-requisite
- Have `etcdctl` installed
```bash
$ sudo apt update; apt install -y etcd-client
# Following that we can perform a snapshot backup of the etcd datastore
$ etcdctl snapshot save snapshotdb --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/server.key
# ... output omitted
Snapshot saved at snapshotdb
```
The above command creates a snapshot in the current directory called `snapshotdb`.
