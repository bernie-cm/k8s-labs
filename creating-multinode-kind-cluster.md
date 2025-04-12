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
