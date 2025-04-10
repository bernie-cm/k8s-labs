# Building a Kubernetes Cluster from Scratch: Part 2 - Installing Container Runtime and Kubernetes Components
This is the second part of my Kubernetes installation series using `kubeadm`. In [Part 1](), I covered the environment preparation, including hardware requirements and network configuration, as well as required Linux kernel modules.

In this Part 2, I'll continue by installing and configuring `containerd` as my container runtime and the installation of the essential Kubernetes componets like kubeadm, kubelet, and kubectl.

## Installing the container runtime
Kubernetes is a container orchestration system, and as such it needs a **container runtime** responsible for running the containers in the cluster. The container runtime pulls the images, starts/stops the containers and report container status back to Kubernetes.

Before installing the container runtime, we need to download the GPG keys to ensure the packages haven't been tampered with, and `apt` will need this information later as well.

```bash
root@cp:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| gpg --dearmor -o /etc/apt/keyrings/docker.gpg
root@cp:~#
root@cp:~# echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Now we install containerd
```bash
root@cp:~# apt-get update && apt-get install containerd.io -y
root@cp:~# containerd config default | tee /etc/containerd/config.toml
root@cp:~# systemctl restart containerd 
```

## Installing the Kubernetes software
First, like with containerd, we need to download the public key for the package repositories.
```bash
root@cp:~# curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the appropriate k8s repository
root@cp:~# echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" \
| tee /etc/apt/sources.list.d/kubernetes.list
```

Finally, update the repo and install the Kubernetes software
```bash
root@cp:~# apt-get update
root@cp:~# apt-get install -y kubeadm=1.31.1-1.1 kubelet=1.31.1-1.1 kubectl=1.31.1-1.1
```

In the next part of this series, I will set the local DNS alias for my `cp` Kubernetes server and assign it with the name `k8scp`. Then I will create a configuration file for the cluster.
