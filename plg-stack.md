# Installing observability stack on Kubernetes
## Using Promtail + Loki + Grafana

- Promtail collects logs from your applications and sends them to Loki
- Loki aggregates and indexes the logs
- Grafana is used to visualise and create dashboards

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

### Deploying Loki
We need to deploy Loki first to ensure there's a place where logs get sent to. We could install Promtail first, but then it won't have a destination for the logs.

```bash
$ helm install loki grafana/loki-stack \
 --namespace=monitoring \
 --create-namespace \
 --set promtail.enabled=false
 --set grafana.enabled=false
 ```
 We disable both Promtail and Grafana installations so we can deploy them later. This command also creates a new namespace to keep resources organised.
