# How to install a Kubernetes cluster
There are multiple tools to install Kubernetes. A community supported tool is `kubeadm` and this lab uses it to install and build a Kubernetes cluster.

## Connecting to future control plane and worker nodes
I'm using AWS to launch two instances that will be my Kubernetes cluster. Both instances are Ubuntu 24.04 with 2vCPUs and 8GB of RAM. Using my AWS key pair, the next step is to connect to both instances using `ssh`.

### Installing Kubernetes
The first step is to connect to the first EC2 instance to update and upgrade the control plane or **cp**.
```bash
$ ssh -i /path/to/my-key.pem ubuntu@instance-ip
$ sudo -i
root@cp:~# apt update && apt upgrade -y
```

With the system updated, the next step is to install a **container runtime**. A commonly used option, easy to deploy and lightweight is `containerd`. Also, Kubernetes has deprecated Docker engine as of 2020, and `containerd` offers simplicity and focus as it doesn't include the additional stuff that Docker includes. This is purely a container runtime which supports the Container Runtime Interface or CRI as expected by Kubernetes.

