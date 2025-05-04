# Expanded Kubernetes Networking Study Notes

## Linux Networking Basics

### Switching
- Devices on the same network can communicate directly
- Use `ip link` to show network interfaces
- Use `ip addr add <ip-address/cidr> dev <interface>` to assign IP addresses
- MAC addresses facilitate communication at Layer 2
- Switches maintain MAC address tables to forward traffic efficiently

### Routing
- Connects different networks together
- Use `route` or `ip route` to view routing table
- Add routes with `ip route add <network-cidr> via <gateway-ip>`
- Enable packet forwarding with `echo 1 > /proc/sys/net/ipv4/ip_forward`
- Make forwarding persistent with `/etc/sysctl.conf` setting
- Default routes (0.0.0.0/0) direct traffic to unknown destinations

### NAT (Network Address Translation)
- Allows private networks to access external networks
- Configure with `iptables -t nat -A POSTROUTING -s <source-network> -j MASQUERADE`
- Source NAT (SNAT) modifies the source address of outgoing packets
- Destination NAT (DNAT) modifies the destination address of incoming packets
- Port forwarding is implemented using DNAT

## DNS Fundamentals

### Basic Concepts
- DNS resolves hostnames to IP addresses
- DNS hierarchy: root → TLD (.com, .org) → domain → subdomain
- Record types: 
  - A (IPv4 address)
  - AAAA (IPv6 address)
  - CNAME (alias/canonical name)
  - SRV (service records)
  - MX (mail exchange)
  - TXT (text records, used for verification)

### DNS Configuration
- Local DNS configuration in `/etc/hosts`
- DNS server configuration in `/etc/resolv.conf`
- Search domains allow shorthand names (set with `search <domain>`)
- Order of resolution defined in `/etc/nsswitch.conf` (e.g., "hosts: files dns")
- Multiple nameservers provide redundancy
- TTL (Time To Live) controls caching duration

### DNS Tools
- `nslookup` - Simple DNS query tool
- `dig` - More detailed DNS query information
- `host` - Simple hostname to IP resolution
- `resolvectl` - DNS resolver query tool (systemd systems)

## Container Networking

### Network Namespaces
- Isolated network stack (interfaces, routing tables, firewall rules)
- Create with `ip netns add <namespace-name>`
- Execute commands in namespace with `ip netns exec <namespace> <command>`
- Short form: `ip -n <namespace> <command>`
- Connect namespaces with virtual ethernet (veth) pairs
- Each namespace has its own loopback interface

### Virtual Ethernet Pairs
- Virtual "cable" connecting network namespaces
- Create with `ip link add <veth1> type veth peer name <veth2>`
- Move to namespace with `ip link set <veth> netns <namespace>`
- Must be brought up with `ip link set <interface> up`
- Names often use convention: `veth<name>` and `veth<name>-br`

### Linux Bridge
- Software switch connecting multiple network segments
- Create with `ip link add <bridge-name> type bridge`
- Connect interfaces to bridge with `ip link set <interface> master <bridge-name>`
- Assign IP to bridge for routing: `ip addr add <ip/cidr> dev <bridge>`
- Bridge acts as default gateway for connected networks
- STP (Spanning Tree Protocol) prevents loops

### Docker Networking

#### Network Types
1. **None** - No network connectivity (`--network none`)
2. **Host** - Container shares host's network namespace (`--network host`)
3. **Bridge** - Default, uses internal bridge network (typically 172.17.0.0/16)
4. **Overlay** - Multi-host networking
5. **Macvlan** - Assigns MAC address to container
6. **IPvlan** - Shares host interface but with unique IP

#### Under the Hood
- Creates network namespaces for containers
- Sets up veth pairs to connect containers to bridge (docker0)
- Configures NAT for external connectivity
- Uses iptables for port mapping
- Creates DNS configurations for container name resolution

## CNI (Container Network Interface)

### Core Concepts
- Standardized interface for network configuration in containers
- Handles creation of network interfaces in container namespaces
- Manages IP address assignment
- Supports ADD/DEL/CHECK operations
- Plugins must implement specific JSON response format

### CNI Plugins
- **Bridge** - Creates Linux bridge networks
- **IPAM** - IP address management:
  - host-local (local file-based IP allocation)
  - DHCP (dynamic addressing)
- **Weave** - Creates mesh network between nodes
- **Flannel** - Layer 3 network fabric with multiple backends
- **Calico** - Layer 3 approach with BGP for routing
- **Cilium** - eBPF-based networking with advanced security
- **Multus** - Enables attaching multiple interfaces to pods

### IPAM (IP Address Management)
- Responsible for allocating and managing IP addresses
- Must avoid conflicts between nodes
- Methods:
  - File-based tracking
  - Central database
  - DHCP
  - Operator-defined ranges

## Kubernetes Networking

### Node Networking
- Each node needs unique IP and hostname
- Key ports:
  - Master: 6443 (kube-api), 2379-2380 (etcd), 10250-10252 (kubelet/scheduler/controller)
  - Worker: 10250 (kubelet), 30000-32767 (NodePort services)
- Nodes must be able to reach each other via IP
- Control plane components may be distributed across multiple nodes
- Cloud providers often handle node networking details

### Pod Networking Model
1. Every Pod gets its own IP address
2. Pods on same node can communicate with each other
3. Pods on different nodes can communicate without NAT
4. Agents on nodes can communicate with all pods
5. Pod IPs must be routable within the cluster network

### Pod-to-Pod Communication
- Implemented by CNI plugins
- Typically uses one of:
  - Overlay networks (VXLAN, Geneve, etc.)
  - BGP route distribution
  - Cloud provider native networking
- Each node has a CIDR block allocated for pods
- Inter-node pod communication depends on CNI implementation:
  - Direct routing between node networks
  - Tunneling/encapsulation (overlay)
  - Policy routing

### Container-to-Container Communication
- Containers in same pod share:
  - Network namespace
  - IP address
  - Port space
  - Loopback interface (localhost)
- Containers in same pod communicate via localhost
- Container ports can conflict if both try to use same port number

### Service Networking

#### Service Types
1. **ClusterIP** - Internal-only service (default)
2. **NodePort** - Exposes service on node IP at static port (30000-32767)
3. **LoadBalancer** - Integrates with cloud provider load balancer
4. **ExternalName** - Maps service to DNS name
5. **Headless** - No ClusterIP, for direct pod connections (returns all pod IPs)

#### Implementation
- ClusterIP creates virtual IP for service in cluster
- Virtual IP is purely iptables/ipvs based
- Service IP range configured with `--service-cluster-ip-range` (default: 10.0.0.0/24)
- Implementation uses:
  - **iptables** - NAT rules to redirect traffic
  - **IPVS** - Kernel-based load balancing (better performance at scale)
  - **Userspace** - Older method, kube-proxy intercepts traffic

#### Kube-proxy Modes
- **iptables** - Default mode, uses NAT rules for redirection
- **ipvs** - More efficient for large clusters, supports more load balancing algorithms
- **userspace** - Legacy mode, slower but more compatible

### DNS in Kubernetes

#### CoreDNS
- Default DNS provider for Kubernetes
- Highly customizable through Corefile
- Runs as pods in kube-system namespace
- Auto-registers services and pods

#### Service Discovery
- Naming patterns:
  - `service-name` - Within same namespace
  - `service-name.namespace` - Cross-namespace
  - `service-name.namespace.svc.cluster.local` - FQDN
- Pod DNS records: `pod-ip-with-dashes.namespace.pod.cluster.local`
- Hostname and subdomain can be set in pod spec
- Configured through kubelet settings (resolv.conf)
- A/AAAA and SRV records for regular and headless services

#### Advanced DNS Features
- Pods inherit DNS configuration from node
- Custom DNS policies available:
  - Default
  - ClusterFirst (default)
  - ClusterFirstWithHostNet
  - None (custom DNS)
- DNS autoscaling based on cluster size

## Network Policies

### Concepts
- Similar to firewalls for pod communication
- Default: all pods can communicate with all pods
- Namespace isolation requires explicit policies
- Based on labels and selectors
- Requires network plugin with NetworkPolicy support

### Policy Types
- **Ingress** - Incoming traffic rules
- **Egress** - Outgoing traffic rules
- Can be combined in single policy

### Selectors
- **podSelector** - Which pods policy applies to
- **namespaceSelector** - Which namespaces traffic can come from/go to
- **ipBlock** - CIDR ranges for external traffic

### Implementation
- Not enforced by Kubernetes itself
- Enforcement depends on CNI plugin:
  - Calico, Cilium, Weave support network policies
  - Flannel does not (requires additional components)

## Ingress

### Components
1. **Ingress Controller** - Implementation (NGINX, Traefik, etc.)
2. **Ingress Resources** - Rules for routing external traffic

### Ingress Controller Setup
- Deployment: Runs the ingress controller pods
- ConfigMap: Controller configuration
- Service: Exposes controller to traffic
- ServiceAccount, Roles, Role Bindings: RBAC configuration
- DaemonSet: Alternative to Deployment, ensures controller on each node
- Annotations: Controller-specific configuration

### Ingress Resources
- Define routing rules based on:
  - Host-based routing (`host: wear.my-online-store.com`)
  - Path-based routing (`path: /wear`)
- Support for default backends
- TLS termination and certificate configuration
- Rewrite and redirect rules (controller-specific)
- Session affinity configurations

### Advanced Ingress Features
- Rate limiting
- Authentication
- URL rewriting
- Custom headers
- SSL/TLS termination
- Webhooks
- Canary deployments
- Cross-Origin Resource Sharing (CORS)

### Ingress YAML Example
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: my-online-store.com
    http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port:
              number: 80
      - path: /watch
        pathType: Prefix
        backend:
          service:
            name: watch-service
            port:
              number: 80
  tls:
  - hosts:
    - my-online-store.com
    secretName: my-tls-secret
```

## Advanced Networking Topics

### Multus CNI
- Enables multiple network interfaces in pods
- Useful for:
  - Network separation
  - Performance optimization
  - Integration with specialized hardware
- Each interface can use different CNI plugin
- Defines primary and secondary interfaces

### Service Mesh
- Abstracts application networking
- Provides:
  - Service discovery
  - Load balancing
  - Encryption (mTLS)
  - Observability
  - Traffic control
- Examples: Istio, Linkerd, Consul
- Typically uses sidecar container model

### IPv6 Support
- Kubernetes supports dual-stack networking
- Requires CNI plugin with IPv6 support
- Services can have IPv4 and IPv6 addresses
- Configuration involves multiple CIDR ranges
- Not all features have complete IPv6 support

### Cloud Provider Integration
- Cloud-specific networking implementations
- LoadBalancer services integrate with cloud load balancers
- Node networking often handled by cloud VPC
- Many cloud providers have their own CNI implementations
- Kubernetes knows how to create cloud resources via cloud controller

### eBPF-based Networking
- Next-generation networking technology
- Enables programmable packet processing in kernel
- Used by CNI plugins like Cilium
- Benefits:
  - Higher performance
  - More flexible policy enforcement
  - Greater observability
  - L7/application-aware networking

### Bandwidth Management
- ResourceQuota can limit total namespace bandwidth
- QoS classes affect network priority
- CNI plugins may support:
  - Bandwidth limiting
  - Traffic shaping
  - Priority queuing

This expanded guide covers more advanced topics in Kubernetes networking, providing a comprehensive reference for both basic and complex networking scenarios.
