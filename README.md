# HAProxy Load Balancer Setup

## 📖 Introduction

HAProxy is used as a TCP load balancer to distribute Kubernetes API traffic across multiple control plane nodes. This improves **high availability** and **fault tolerance** of your Kubernetes cluster in an air-gapped environment.

---

## 📦 Installation (Air-Gapped)

### 1. Download HAProxy RPM on a machine with internet access

```bash
yum install --downloadonly --downloaddir=. haproxy

```
### 2.  Transfer RPM to HAProxy node 

Use scp or USB to copy the .rpm file to your HAProxy host.

### 3. Install RPM on the HAProxy node

```bash

rpm -ivh haproxy-*.rpm

```
⚙️ Configuration

### 1. Edit HAProxy config file  

Edit at /etc/haproxy/haproxy.cfg   use the github file in the folder

### 2. Restart and Enable HAProxy

```bash

systemctl restart haproxy
systemctl enable haproxy

```


## 🚀 Kubernetes Air-Gapped HA Cluster Setup

This repository contains the complete setup and documentation for deploying a **Kubernetes HA cluster in an air-gapped environment**.

### Prerequsite:

- ✅ 3 Control plane nodes
- ✅ 1 Worker node
- ✅ HAProxy load balancer
- ✅ Private Docker registry (JFrog Artifactory with SSL)
- ✅ Offline image management
- ✅ kubeadm-based cluster initialization

---

### 🖥️ Kubernetes Node Setup (Air-Gapped Environment)

> 🔁 These steps must be performed on **all control plane and worker nodes**.


## 📌 1. Disable SELinux and Firewall

Kubernetes requires these to be disabled for consistent networking and container behavior.

```bash
# Disable SELinux
sudo setenforce 0

# Disable and stop the firewall
sudo systemctl stop firewalld
sudo systemctl disable firewalld

```

### 2. Add Kubernetes YUM Repository (On Internet machine)

```bash

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/repodata/repomd.xml.key
EOF
```

### 3. Download Required RPM Packages (Internet Machine)

```bash

# Create a directory to store packages
mkdir k8s-rpms && cd k8s-rpms

# Download Kubernetes components
yum install --downloadonly --downloaddir=. kubeadm kubelet kubectl

# Download container runtime (e.g., containerd)
yum install --downloadonly --downloaddir=. containerd

🎯 These RPMs can now be copied to all your air-gapped nodes.
```

### 4. Install RPMs on Air-Gapped Nodes

```bash

cd /path/to/rpms/

sudo yum localinstall *.rpm --disablerepo="*"

```

### 5. Disable Swap
```bash

# Temporarily disable swap
sudo swapoff -a

# Permanently disable swap on reboot
sudo sed -i '/ swap / s/^/#/' /etc/fstab

```
### 6. Load Kernel Modules and Set Sysctl Params

Kubernetes networking requires certain kernel modules and sysctl settings:

```bash
# Enable br_netfilter module at boot
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

# Apply module now
sudo modprobe br_netfilter

# Set required sysctl parameters
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply settings
sudo sysctl --system

```

### 7. Configure Containerd

Containerd must be configured to trust the private registry (especially when using self-signed SSL certs).

```bash

Step 1: Create cert directory for registry

sudo mkdir -p /etc/containerd/certs.d/192.168.95.35:9443

# Copy the self-signed certificate

sudo cp /path/to/registry.crt /etc/containerd/certs.d/192.168.95.35:9443/ca.crt

Step 2: Add Registry Certificate to System Trust Store

sudo cp /etc/containerd/certs.d/192.168.95.35:9443/ca.crt  /etc/pki/ca-trust/source/anchors/registry.crt

sudo update-ca-trust extract


2.	Edit /etc/containerd/config.toml and apply the full configuration including:
   a.	Custom registry endpoint
   b.	Authentication
   c.	TLS CA file for containerd

Restart containerd

systemctl restart containerd

Ensures containerd can pull images securely over HTTPS from your internal registry.   (Perform all these steps on all nodes)

```








