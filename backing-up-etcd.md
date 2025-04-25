# Backing up and restoring the etcd datastore

### How to perform a backup of the etcd datastore
To back up the cluster store, or `etcd`, we can create a snapshot file using the CLI tool `etcdctl`. This lab assumes a successful installation of the `etcdctl` tool and that prior knowledge of what `etcd` is and its purpose exists.

```bash
# First perform the backup with snapshot option 
$ etcdctl snapshot save etcd-backup --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.crt

# ... Output omitted
Snapshot saved at etcd-backup
```
`--cacert` verifies certificates using the k8s Certificate Authority (CA)
`--cert` identifies secure client using the `etcd` server certificate
`--key` identifies secure client using the `etcd` key file

### Restoring the etcd backup

To restore the backup we use again the `etcdctl` CLI tool and the `snapshot` command. What's key in this task is that the backup will be **restored to an `etcd` directory**. That's why we use the `--data-dir` option with the command.

```bash
# Second perform the restore operation
# The command here will restore the backup to the /var/lib/etcd-backup directory
$ etcdctl snapshot restore etcd-backup --data-dir /var/lib/etcd-backup

# ... Output omitted
```
