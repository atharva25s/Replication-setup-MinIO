# Filecloud integration with MinIO

# Overview
- FileCloud is a powerful, self-hosted file-sharing and collaboration platform designed for businesses and enterprises. 
- It offers secure file storage, sharing, sync, and access control, ensuring data privacy and compliance with industry standards.
- Mobile & Desktop Access: Apps for iOS, Android, Windows, and macOS.

# Prerequisites
- Go to [Community Edition of Filecloud](https://ce.filecloud.com/) and register yourself.
- After registration go to [Download Server](https://portal.getfilecloud.com/ui/user/index.html#/sites/trial/community).
- Generate the License and download it.

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
- Look for a piece of code where 
```bash
// Storage implementation 'local' or 'openstack' or 'amazons3'
define("TONIDOCLOUD_STORAGE_IMPLEMENTATION", "local");
```
- Change it to
```bash
define("TONIDOCLOUD_STORAGE_IMPLEMENTATION", "amazons3");
```
- Press **Esc**, then type **:wq** to save and exit.

#### Set the amazons3storageconfig
- Run this command in the same directory
```bash
cp amazons3storageconfig-sample.php amazons3storageconfig.php
```
## Access Key generation on MinIO Dashboard
- Go to the console of MinIO of Node 1.
- Access Key -> Create access key.
- Enable JSON Policy and copy
```bash
{

    "Version": "2012-10-17",

    "Statement": [

                    {

                        "Effect": "Allow",

                        "Action": [

                                "s3:CreateBucket",

                                "s3:DeleteObject",

                                "s3:GetObject",

                                "s3:ListBucket",

                                "s3:PutObject"

                        ],

                        "Resource": [

                                "arn:aws:s3:::bucketname/*"

                        ]

                    }

                ]

}
```
- Change the bucket name to *filecloud-data*
- Create and Download the Access_key credentials

## MinIO setup on Filecloud Dashboard
- Filecloud admin -> Settings -> Storage.
- You will have storage settings enabled for S3.
- Fill in the required details
```bash
Access_key : <Your access key>
Secret_key : <Your secret key>
S3_bucket_name : <Your bucket name>
URL_endpoint : <http://NODE1_PRIVATE_IP:9000>
```
- Save it.

## User creation
- Filecloud admin -> Users -> Create user
- After creating the user, Login via user credentials on 
```bash
http://<NODE1_PUBLIC_IP>
```
- Let's hope for the best.

