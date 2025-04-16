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
To check the status of the snapshot you can use `$ etcdctl snapshot status snapshotdb --write-out=table`. This will generate a table like below
```
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| e4f4157c |   102366 |        817 |     2.2 MB |
+----------+----------+------------+------------+
```
## Restoring etcd from snapshot
If needed, we can use etcdctl to also restore the backed up version of the datastore. In order to perform the restore operation, we need to specify a directory already accessed by etcd. By checking the `etcd.yaml` file in the `/etc/kubernetes/manifests` directory, we can change that directory to something like `/var/lib/etcd-restore`.
```bash
$ etcdctl snapshot restore <snapshot-name> --data-dir /var/lib/etcd-restore
```

## Upgrading Kubernetes version
The last task we will perform in this lab is upgrading the version of `kubeadm`. For example, if in the exam there's a question about a company needing to upgrade the Kubernetes controller to version X, then we know that `kubeadm` is the tool to use.
The first thing to do is to view the version of our control plane components.
```bash
# This will show a list of control plane components and their current version
# as well as the version to which we can upgrade
$ kubeadm upgrade plan

COMPONENT                 NODE                      CURRENT    TARGET
kube-apiserver            acing-cka-control-plane   v1.32.2    v1.32.3
kube-controller-manager   acing-cka-control-plane   v1.32.2    v1.32.3
kube-scheduler            acing-cka-control-plane   v1.32.2    v1.32.3
kube-proxy                                          1.32.2     v1.32.3
CoreDNS                                             v1.11.3    v1.11.3
etcd                      acing-cka-control-plane   3.5.16-0   3.5.16-0
```
After running the command above you will get the command needed to upgrade, as output.
`$ kubeadm upgrade apply v1.32.3`

