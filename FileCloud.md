# Filecloud integration with MinIO

# Overview
- FileCloud is a powerful, self-hosted file-sharing and collaboration platform designed for businesses and enterprises. 
- It offers secure file storage, sharing, sync, and access control, ensuring data privacy and compliance with industry standards.
- Mobile & Desktop Access: Apps for iOS, Android, Windows, and macOS.

# Prerequisites
- Go to [Community Edition of Filecloud](https://ce.filecloud.com/) and register yourself.
- After registration go to [Download Server](https://portal.getfilecloud.com/ui/user/index.html#/sites/trial/community).
- Download server for Docker.
- Also generate the License and download it.

# Setup
## On **NODE 1**
#### Pull the docker image of Filecloud
```bash
docker pull filecloud/fileclouddocker:23.241
```

#### Run the container
```bash
sudo docker run --privileged -d -p 443:443 -p 80:80 -v fcdata:/opt/fileclouddata -v dbdata:/var/lib/mongodb -v solrdata:/opt/solrfcdata/var/solr -v htmldata:/var/www/html --name <yourcontainername> <current_image_name:tag> /lib/systemd/systemd
```

#### Access the **Admin Portal**
```bash
http://<PUBLIC_IP_NODE_1>/ui/admin/index.html
```
- Username: admin
- Password: password

#### Attaching the License
- Once you enter the Filecloud console, you will asked to upload the license. Upload the *License.xml* that you have downloaded earlier

## Setting up the MinIO storage in filecloud container
- **Note**: Filecloud understands any object storage as *Amazon S3*. So here MinIO == amazonS3

#### Enter into the Docker environment of Filecloud container
```bash
docker exec -it <container-name> /bin/bash
```
**or**
```bash
docker exec -it <container-name> /bin/sh
```
#### Change the directory
```bash
cd /var/www/html/config
```

#### Open the file
```bash
vi cloudconfig.php
```
- Press **i** to enter insert mode.
- 
- Press **Esc**, then type **:wq** to save and exit.

#### Set the amazons3storageconfig
- Run this command in the same directory
```bash
cp
```
## Access Key generation on MinIO Dashboard

## MinIO setup on Filecloud Dashboard

## User creation

