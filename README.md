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

# 🖥️ Kubernetes Node Setup (Air-Gapped Environment)

> 🔁 These steps must be performed on **all control plane and worker nodes**.

---

## 📌 1. Disable SELinux and Firewall

Kubernetes requires these to be disabled for consistent networking and container behavior.

```bash
# Disable SELinux
sudo setenforce 0

# Disable and stop the firewall
sudo systemctl stop firewalld
sudo systemctl disable firewalld

```


