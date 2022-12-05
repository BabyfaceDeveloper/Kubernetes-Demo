# Kubernetes demonstration (Installation, Deployment, Updating)

> Algonquin College final demo for Emerging Technologies project

# Table of Contents

- [Kubernetes demonstration (Installation, Deployment, Updating)](#kubernetes-demonstration-installation-deployment-updating)
- [Table of Contents](#table-of-contents)
  - [Description](#description)
  - [Getting Started](#getting-started)
    - [Dependencies](#dependencies)
    - [Installing Kubernetes](#installing-kubernetes)
    - [Installing Kubernetes Dashboard](#installing-kubernetes-dashboard)
    - [Deploy a Deployment or Service](#deploy-a-deployment-or-service)
  - [Authors](#authors)
  - [Version History](#version-history)
  - [License](#license)


## Description

This repo contains Installation, Deployment and Updating sections to create a Kubernetes cluster and deploy an application to cluster

## Getting Started

### Dependencies

* Ubuntu 20.x or above (Server or Desktop)

### Installing Kubernetes

1. **Tools**
  * Install SSH and nano
```bash
sudo apt install nano openssh-server -y
```
2. **Prerequisite setup**
  * Change the ip address
```bash
sudo nano /etc/netplan/.... # the file of netplan start with 00- or 99-
```
  * Change hostname
```bash
sudo hostnamectl hostname 'your host name here'
```
  * Change hostname of `hosts` files
```bash
sudo nano /etc/hosts # change old hostname to new one
```
  * Reboot machine
```bash
sudo reboot now
```
3. **Disable swap volume**
```bash
sudo swapoff -a && sudo sed 's;^.*swap;#&;g' -i /etc/fstab
```
4. **Install containerd (Docker runtime)**
```bash
sudo apt update && sudo apt -y install \
ca-certificates \
curl \
gnupg \
lsb-release
```
```bash
sudo mkdir -p /etc/apt/keyrings && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
```bash
# add docker apt to apt source
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```bash
sudo apt-get update && sudo apt-get install containerd.io -y
```
```bash
# Exports config files used by kubelet and systemd cgroup
sudo mkdir -p /etc/containerd && containerd config default | sudo tee /etc/containerd/config.toml
```
5. **Configuring a cgroup driver**
```bash
sudo sed -i "s/SystemdCgroup = false/SystemdCgroup = true/g" /etc/containerd/config.toml && sudo systemctl restart containerd
```
6. **Forwarding IPv4 and letting iptables see bridged traffic (allows for nodes to join control plane)**
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay && sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```
7. **Install kubeadm, kubelet and kubectl**
```bash
sudo apt-get update 
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl # sync update between kubeadm, kubelet and kubectl
```
8. **Create network inside kubernetes cluster (only for control plane)**
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 # Calico network
```
9. **Configure kubectl (only for control plane)**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
10. **Join node(s) to control plane (only for nodes vm or server)**
```bash
sudo kubeadm join 'ip address of control plane':6443 --token 'token here' --discovery-token-ca-cert-hash 'sha hash here'
```

* Forget the kubeadm join command ?
```bash
sudo kubeadm token create --print-join-command
```

### Installing Kubernetes Dashboard

### Deploy a Deployment or Service

```bash
kubectl apply -f 'your file here'
```

## Authors

Contributors names and contact info

[@Nguyen](https://www.linkedin.com/in/binhnguyennguyen/)

## Version History

* 0.1
    * Initial Release

## License

This repo is licensed under the MIT License - see the LICENSE.md file for details
