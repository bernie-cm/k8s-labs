# How to install a Kubernetes cluster – Part 1
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

We need to install some dependencies.
```bash
root@cp:~# apt install apt-transport-https software-properties-common ca-certificates socat -y
```

Next, two kernel modules need to be loaded due to Kubernetes infrastructure requirements. They enable core container and networking functionality that Kubernetes needs. The `overlay` module relates to storage management and the `br_netfilter` module relates to networking and packet routing.
```bash
root@cp:˜# modprobe overlay
root@cp:˜# modprobe br_netfilter
```

Finally, we need to tweak the network routing and policies as Kubernetes relies on `iptables` rules. Without creating the rules below, network packets may bypass `iptables` rules and inter-pod communication **would fail**.
The below command creates a configuration file `kubernetes.conf` at `/etc/sysctl.d` and it's loaded when the system starts. The two lines below, starting with `net.bridge…` ensure that IPv4 and IPv6 packets follow `iptables` rules. Finally, `ip_forward` enables IP forwarding in the Linux kernel.

```bash
root@cp:˜# cat << EOF | tee /etc/sysctl.d/kubernetes.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> net.ipv4.ip_forward = 1
EOF
```
Once those rules have been added, we ensure the changes are loaded by the current kernel.
```bash
root@cp:~# sysctl --system
```
That's part 1 on this series. The next part will deal with the installation of the `containerd` runtime and the Kubernetes software itself.
