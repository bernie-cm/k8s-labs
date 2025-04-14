# Why to use --dry-run when deploying

To save time, and ensure the syntax and formatting are correct, a more efficient way to create a YAML file with the specs for a pod is to use `kubectl` and the `--dry-run` flag.

```bash
$ kubectl run pod --image nginx --dry-run=client -o yaml > nginx-pod.yaml
```

This will let `kubectl` write the YAML file. The main benefits of doing it this way are:
1. You can modify the YAML template *before* having to deployt it to the Kubernetes cluster.
2. You can modify the template to add volumes, env variables and other configurations.
3. The formatting of the YAML file will be correct and you reduce the chances of syntax error.
