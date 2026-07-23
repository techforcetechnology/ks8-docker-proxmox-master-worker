# ks8-docker-proxmox-master-worker
## kubernetees selfhosted server install on proxmox
# Hardware Recommendation
## **Control Plane**

4 vCPU \
8 GB RAM \
50 GB SSD

## Worker
4 vCPU \
8 GB RAM \
80 GB SSD
# Minimum cluster

| Node     | IP |
|-----      |-----------|
|master     |192.168.10.10|
|worker1    |192.168.10.11|
|worker2    |192.168.10.12|


**Ubuntu Server 24.04 LTS or newer**

# Step 1 Install Proxmox

Install latest Proxmox VE
# Step 2 Create Ubuntu VM Template

## Install Ubuntu
Install
```
sudo apt update 
sudo apt install openssh-server qemu-guest-agent suocurl wget vim git
```
## Shutdown

Convert to Template

Clone
```
master

worker1

worker2

worker3
```

| Node     | IP |
|-----      |-----------|
|master     |192.168.10.10|
|worker1    |192.168.10.11|
|worker2    |192.168.10.12|

## Update hosts
```
nano /etc/hosts
```

# Step 4 Disable Swap
```
swapoff -a
```
Edit
```
/etc/fstab
```
**Comment swap**
Verify
```
free -h
```
## Step 5 Kernel Modules

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

Load

```
modprobe overlay
modprobe br_netfilter
```
## Step 6 Sysctl
```
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf

net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward=1

EOF

```

Apply
```
sysctl --system
```
## Step 7 Install Docker
```
apt update
apt install ca-certificates curl gnupg lsb-release
```
### Install Docker
```
curl -fsSL https://get.docker.com | sh
```
Enable
```
systemctl enable docker
systemctl start docker
```
## Step 8 Install cri-dockerd
Docker requires CRI.
Clone the repository
```
git clone https://github.com/Mirantis/cri-dockerd.git

```
Enter the directory
```
cd cri-dockerd

```
## Build cri-dockerd
Compile the binary

```
mkdir -p bin
go build -o bin/cri-dockerd
```
**wait until it will finish the process**
After compilation, verify the binary exists
```
ls -l bin/
```
You should see
```
cri-dockerd
```

## Install the Binary
Copy it into your PATH

```
sudo install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd

```
Verify
```
cri-dockerd --version
```
## Install the systemd Service

**Copy the service files**
```
sudo cp packaging/systemd/* /etc/systemd/system/
```
Update the binary path in the service file
```
sudo sed -i 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' \
/etc/systemd/system/cri-docker.service
```
**Reload systemd**

```
sudo systemctl daemon-reload
```
## Enable the Service
```
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
```
Check status
```
sudo systemctl status cri-docker

```
**You should see**
## Active: active (running)


## Verify the Socket

```
ls -l /var/run/cri-dockerd.sock

```
or
```
ls -l /run/cri-dockerd.sock
```
**You should see a Unix socket file**
**View existing logs**
```
sudo journalctl -u cri-docker --no-pager
```
or
```
sudo journalctl -xeu cri-docker
```
**These commands will display existing logs and exit**

**Verify the service is running**
```
sudo systemctl is-active cri-docker
```
**Expected output**
active

## Step 9 Install Kubernetes
```
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg
```
## Create the keyrings directory
```
sudo mkdir -p /etc/apt/keyrings
```
## Download the Kubernetes signing key
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
## Add the Kubernetes repository
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```
## Update package lists and Install Kubernetes
```
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
```
## Prevent automatic upgrades
```
sudo apt-mark hold kubelet kubeadm kubectl
```
## Verify the installation
Run
```
kubeadm version
kubectl version --client
kubelet --version
```
## Use cri-dockerd with kubeadm

**When initializing the control plane**
```
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --cri-socket=unix:///var/run/cri-dockerd.sock
```
**--cri-socket=unix:///var/run/cri-dockerd.sock** This will not added in the token so please add this line in the token

## When joining worker nodes
```
sudo kubeadm join <MASTER_IP>:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --cri-socket=unix:///var/run/cri-dockerd.sock
```
**--cri-socket=unix:///var/run/cri-dockerd.sock** This will not added in the token so please add this line in the token

## Verify Everything
Run
```
sudo systemctl status docker
sudo systemctl status cri-docker
docker ps
```
Expected output
```
Docker:        active (running)
cri-docker:    active (running)
```



















































