# 2 Node Replication Setup with MinIO

# Overview

## MinIO
- MinIO is an open-source, high-performance, and scalable object storage system designed for cloud-native applications.
- MinIO is commonly used for storing large amounts of unstructured data, such as backups, logs, machine learning datasets, and media files.

## Distributed Setup
- This setup of MinIO helps in Replication by acheiving redundancy.
- **Main node** and **Replica** node are always in sync.
- Failure of **Main node** can be restored using the **Replica Node**.

# Requirements

## Hardware (Physical PC)

#### Main Node (Node 1)
  - OS: 22.04/24.04 LTS Ubuntu
  - RAM: 8/16 GB
  - ROM: 1TB (or preferred size)

#### Replica Node (Node 2)
  - OS: 22.04/24.04 LTS Ubuntu
  - RAM: 8/16 GB
  - ROM: 1TB (or preferred size) 

#### Networking
- Ensure that both the nodes are running on same Network
- Obtain Public and Private IPs of Node 1 and Node 2

- Private IP Address:
```bash
ip a | grep inet
```
**OR**  
```bash
hostname -I
```

- Public IP Address:
```bash
curl -s ifconfig.me
```
**OR**  
```bash
wget -qO- ifconfig.me
```

## Software (Virtual Machines)

#### Main Node (Node 1)
  - OS: 22.04/24.04 LTS Ubuntu
  - RAM: 8/16 GB
  - ROM: 1TB (or preferred size)

#### Replica Node (Node 2)
  - OS: 22.04/24.04 LTS Ubuntu
  - RAM: 8/16 GB
  - ROM: 1TB (or preferred size) 

#### Networking
- Ensure that both the machines are present on the same Virtual Private Cloud/Netowrk.
- You will be provided with the Public and Private IP addresses.
- Make sure to allow SSH, HTTP, HTTPS traffic for both nodes.
- Expose ports from range 9000-9090 for both nodes.

# Setup

## Initial setup for **BOTH** nodes

### RUN THESE COMMANDS ON BOTH NODES
#### Update the Both nodes
```bash
sudo apt update
```

#### Install Docker
```bash
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

#### Add user to Docker group
```bash
sudo usermod -aG docker $USER
newgrp docker
```

#### Create a directory for MinIO data
```bash
mkdir -p ~/minio/data
```

#### Run MinIO container
```bash
docker run -d --name minio \
  -p 9000:9000 -p 9001:9001 \
  -e "MINIO_ROOT_USER=admin" \
  -e "MINIO_ROOT_PASSWORD=admin123" \
  -v /data/minio:/data \
  quay.io/minio/minio server /data --console-address ":9001"
```  
#### Verify if the containers are running
```bash
docker ps
```

#### Access the MinIO console on both nodes
- Node 1
``` bash
http://<NODE1_PUBLIC_IP>:9001
```
- Username: admin
- Password: admin123

- Node 2
```bash
http://<NODE2_PUBLIC_IP>:9001
```
- Username: admin   
- Password: admin123

#### Install **mc** CLI for MinIO
```bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc -O mc
chmod +x mc
sudo mv mc /usr/local/bin/
mc --version  
```

## Setting up clients on **BOTH** node


#### Configure aliases **BOTH NODES**
```bash
mc alias set node1 http://<PRIVATE_IP_NODE1>:9000 admin admin123
mc alias set node2 http://<PRIVATE_IP_NODE2>:9000 admin admin123
```

#### Test the connection **BOTH NODES**
```bash
mc admin info node1
mc admin info node2
```

#### Create bucket on **NODE 1**
```bash
mc mb node1/filecloud-data
```

#### Create bucket on **NODE 2**
```bash
mc mb node2/filecloud-data
```

#### Verify the buckets **BOTH NODES**
```bash
mc ls node1
mc ls node2
```

#### Enable versioning on bucket for replication **NODE 1**
```bash
mc version enable node1/filecloud-data
```

#### Enable versioning on bucket for replication **NODE 2**
```bash
mc version enable node2/filecloud-data
```

## Setting up the Replication on **NODE 1**
```bash
mc replicate add node1/filecloud-data \
   --remote-bucket node2/filecloud-data \
   --replicate "delete-marker,delete,existing-objects,metadata-sync"
```


# Testing
- Login into the console of **NODE 1** using *admin* credentials.
- Object Browser -> filecloud-data -> upload file.
- Login into the console of **NODE 1** using *admin* credentials.
- Object Browser -> filecloud-data.
- Check if the file uploaded from **NODE 1** bucket is present in **NODE 2** bucket.

# Keeping the Services Running
- Since we are using Docker, it is not required to run these services as system services using systemd setup
- The following commands are helpful for keeping the MinIO services on both nodes.
#### Enable automatic restart
```bash
docker update --restart=always minio
```
#### Reboot the server
```bash
sudo rebooth
```
#### After rebooting check if MinIO is running
```bash
docker ps
```