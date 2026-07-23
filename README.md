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
