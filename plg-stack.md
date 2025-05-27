# Installing observability stack on Kubernetes
## Using Promtail + Loki + Grafana

### Installing helm (if not available)
```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

With that done, add the grafana repo from helm to your local cluster
```bash
$ helm repo add grafana https://grafana.github.io/helm-charts
$ helm repo update
$ <output omitted>
```
