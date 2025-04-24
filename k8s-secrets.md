# Using Kubernetes Secrets
### What are they?
- Like `ConfigMaps` but they store secret information (sensitive) instead of application configuration data.
- They store sensitive information in Base64 encryption.
- Careful, as they are not super secure because anyone with access to your container can read the secret, so they should not be the only security mechanism

### How are they created?
Just like `ConfigMaps` they can be created using imperative methods like `k create secret generic <secret-name> --from-literal=username=admin --from-literal=password=toor`
They can also be created using a YAML file and a declarative method. For example, as above but add `--dry-run=client -o yaml` and then `k apply -f <secret-file.yaml>` 

### How are they used?
They can be injected into a pod definition file. For example, you first create your pod using `k run secret-pod --image busybox --command "sh" "c" "echo The app is running! && sleep 3600" --dry-run=client -o yaml > secret-pod.yaml`
Once the file is created we can mount the secret file as a directory that can be accessed from inside the container. Open the pod YAML and include
```yaml
spec:
  containers:
  - image: busybox
    name: secret-pod
    volumeMounts:
    - name: secret-vol
      mountPath: "/etc/secret-vol"
  volumes:
  - name: secret-vol
    secret:
      secretName: dev-login

