# Installing Kubernetes - Part 2

## Installing the container runtime
Kubernetes is a container orchestration system, and as such it needs a **container runtime** responsible for running the containers in the cluster. The container runtime pulls the images, starts/stops the containers and report container status back to Kubernetes.

Before installing the container runtime, we need to download the GPG keys to ensure the packages haven't been tampered with, and `apt` will need this information later as well.

```bash
root@cp:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
>  | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
root@cp:~#
root@cp:~# echo \
>  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
>  https://download.docker.com/linux/ubuntu \
>  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```
