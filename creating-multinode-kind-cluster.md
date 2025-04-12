# Using kind to create a multinode cluster

## Defining the YAML file
```bash
$ cat << EOF | tee config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worke
EOF
```

## Using kind to create the cluster
With the `config.yaml` created, use `kind` to create the new cluster.
```bash
$ kind cluster create --config config.yaml
```

## Accessing the new cluster
When deploying a new cluster, `kind` will use Docker to create the new nodes. To access the control plane, for example, you would do so.
```bash
$ docker exec -it kind-control-plane bash
root@kind-control-plane:/# kubectl get no
NAME                 STATUS   ROLES           AGE     VERSION
kind-control-plane   Ready    control-plane   7m3s    v1.32.2
kind-worker          Ready    <none>          6m52s   v1.32.2
kind-worker2         Ready    <none>          6m52s   v1.32.2
```
