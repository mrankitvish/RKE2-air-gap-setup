# RKE2 Air-Gapped Environment Setup

This document provides step-by-step instructions to set up an RKE2 (Rancher Kubernetes Engine 2) cluster in an air-gapped environment. Follow the steps carefully to ensure a smooth installation.

---

## Prerequisites

- All nodes (Master and Worker) must have access to the required RKE2 artifacts.
- Ensure connectivity between Master and Worker nodes for the cluster setup.

---

## Step 1: Add Dummy Route  
**(Perform this step on every Worker and Master node)**

Run the following commands to create a dummy route:

```bash
ip link add dummy0 type dummy
ip link set dummy0 up
ip addr add 203.0.113.254/31 dev dummy0
ip route add default via 203.0.113.255 dev dummy0 metric 1000
```

## Step 2: Check and Set SELinux to Disabled
**(Perform this step on every Worker and Master node)**

## Install policycoreutils:
```bash
sudo apt install policycoreutils -y
```
## Verify SELinux status:
```bash
sestatus
```


## Step 3: RKE2 Installation on Kubernetes Master Nodes
**(Perform these steps on Master nodes)**

- Prepare Directories and Download Artifacts:
```bash
mkdir -p /var/lib/rancher/rke2/agent/images/ && cd /var/lib/rancher/rke2/agent/images/
```
Go to the RKE2 [release](https://github.com/rancher/rke2/releases) page and copy the desired version and update it below.
```bash
export MYRKE2_VERSION=v1.29.10%2Brke2r1
```
```bash
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/rke2-images.linux-amd64.tar.zst
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/rke2-images-core.linux-amd64.tar.zst
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/rke2-images-canal.linux-amd64.tar.zst
```
```bash
mkdir /root/rke2-artifacts && cd /root/rke2-artifacts/
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/rke2.linux-amd64.tar.gz
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/sha256sum-amd64.txt
curl -sfL https://get.rke2.io --output install.sh
cp /var/lib/rancher/rke2/agent/images/rke2-images.linux-amd64.tar.zst /root/rke2-artifacts/
```

- Create the Configuration File:
Create `/etc/rancher/rke2/config.yaml` and add the following content. Replace `<MASTER_NODE_IP>` with the Master node's IP:

```yaml
write-kubeconfig-mode: "0644"
tls-san:
  - "<MASTER_NODE_IP>"
```
- Install and Start RKE2 Server:
```bash
export INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts
chmod +x install.sh
./install.sh
```
```bash
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```
- View Logs
To monitor the RKE2 server logs, run the following in another terminal:

```bash
journalctl -fxu rke2-server.service
```
## Step 4: RKE2 Installation on Kubernetes Worker Nodes
**(Perform these steps on Worker nodes)**

- Prepare Directories and Download Artifacts:
```bash
mkdir -p /var/lib/rancher/rke2/agent/images/ && cd /var/lib/rancher/rke2/agent/images/
```

Use the same RKE2 version as the Master node
```bash
export MYRKE2_VERSION=v1.29.10%2Brke2r1
```
```bash
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/rke2-images.linux-amd64.tar.zst
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/rke2-images-core.linux-amd64.tar.zst
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/rke2-images-canal.linux-amd64.tar.zst

mkdir /root/rke2-artifacts && cd /root/rke2-artifacts/
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/rke2.linux-amd64.tar.gz
wget https://github.com/rancher/rke2/releases/download/$MYRKE2_VERSION/sha256sum-amd64.txt
curl -sfL https://get.rke2.io --output install.sh

cp /var/lib/rancher/rke2/agent/images/rke2-images.linux-amd64.tar.zst /root/rke2-artifacts/
```
- Create the Configuration File:
Create /etc/rancher/rke2/config.yaml and add the following content. Replace `<MASTER_NODE_IP>` with the Master node's IP and `<MASTER_NODE_TOKEN>` with the token found at `/var/lib/rancher/rke2/server/node-token` on the Master node:

```yaml
server: https://<MASTER_NODE_IP>:9345
token: <MASTER_NODE_TOKEN>
```
- Install and Start RKE2 Agent:
```bash
export INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts
chmod +x install.sh
export INSTALL_RKE2_TYPE="agent"
./install.sh agent

sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service
```
- View Logs:
To monitor the RKE2 agent logs, run the following in another terminal:

```bash
journalctl -fxu rke2-agent.service
```
## Adding Additional Nodes
Repeat Step 4 to add more Worker nodes to the cluster.

