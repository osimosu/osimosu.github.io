---
layout: "post"
title:  "How to deploy a simple webapp to AWS Lightsail using Docker and ECR"
date:   "2021-07-04 13:30:00 -0500"
categories: aws, lightsail, docker, docker-compose, ecr, bash
---

This is a quick post to show how to deploy a simple containerized webapp
to [Amazon Lightsail](https://aws.amazon.com/lightsail/){:target="_blank"}
using [Docker](https://www.docker.com/){:target="_blank"}
and [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/){:target="_blank"}.

There are many ways to deploy apps to AWS with different use cases and pricing models, but what if you have a small
webapp you want to rapidly deploy on a cheap monthly plan and quickly scale when needed? Lightsail is an excellent choice! It is
AWS' easy-to-use virtual private server that offers bundles of resources such as compute power, memory and storage for
as low as **$3.5/month**!

ECR is a fully managed container registry that makes it easy to store, manage, share, and deploy your container images.

My hobby project [Trainingpass](http://trainingpass.com){:target="_blank"} is a containerized Java and Angular
application hosted on a $10 instance plan which comes with 2 GB memory, 1 vCPU, 60 GB SSD and 3 TB of data transfer.
More than enough for an hobby project with little traffic!

Let's look how to deploy an `example` docker webapp:

#### Step 1. Create instance

[Create a Lightsail instance](https://lightsail.aws.amazon.com/ls/webapp/home/instances){:target="_blank"} and connect
to the instance. The easiest way to connect is by using the browser-based SSH client that is available in the Lightsail
console.

#### Step 2. Install the latest version of docker

```bash
curl -sSL https://get.docker.com | sh
usermod -aG docker ubuntu
```

#### Step 3. Install the latest version of docker-compose

```bash
COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
curl -L https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

#### Step 4. Install amazon-ecr-credential-helper

[Amazon ECR Docker Credential Helper](https://github.com/awslabs/amazon-ecr-credential-helper){:target="_blank"} is a
credential helper for the Docker daemon that makes it easier to use Amazon Elastic Container Registry.

```bash
apt install amazon-ecr-credential-helper
```

#### Step 5. Configure amazon-ecr-credential-helper

```bash
mkdir /root/.docker
cat <<EOT >> /root/.docker/config.json
{
    "credHelpers": {
        "<aws_account_id>.dkr.ecr.<region>.amazonaws.com": "ecr-login"
    }
}
EOT
```

#### Step 6. Configure aws credentials

```bash
mkdir /root/.aws
cat <<EOT >> /root/.aws/credentials
[default]
aws_access_key_id=aws_access_key_id
aws_secret_access_key=aws_secret_access_key
EOT
```

#### Step 7. Configure aws region

```bash
mkdir /root/.aws
cat <<EOT >> /root/.aws/config
[default]
region=<aws region>
output=json
EOT
```

#### Step 8. Copy your docker-compose file to /srv/docker

```bash
mkdir /srv/docker
cat <<EOT >> /srv/docker/example.yml
version: '3.8'
services:
  example-app:
    image: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/example:latest
    ports:
      - <host port>:<container port>
EOT
```

#### Step 9. Create systemd unit file and register it so compose file runs on system restart

```bash
cat <<EOT >> /etc/systemd/system/example.service
[Unit]
Description=Example Service
Requires=docker.service
After=docker.service

[Service]
Type=simple
RemainAfterExit=yes
# match the below to wherever you copied your docker-compose.yml
WorkingDirectory=/srv/docker
ExecStartPre=/usr/local/bin/docker-compose -f example.yml -p example pull
ExecStart=/usr/local/bin/docker-compose -f example.yml -p example up -d
ExecStop=/usr/local/bin/docker-compose -f example.yml -p example down
TimeoutStartSec=0
User=root

[Install]
WantedBy=multi-user.target
EOT
```

#### Step 10. Enable the service and reboot

```bash
systemctl enable example
reboot
```

#### Step 11. Push your image to ECR and restart the service

```bash
systemctl restart example
```

Done! Your application should available at `http://<instance public ip>:<host post>`!

For [Trainingpass](http://trainingpass.com){:target="_blank"}, I've also
setup [Nginx](https://www.nginx.com/blog/setting-up-nginx/){:target="_blank"} and secured it
with [Let's Encrypt SSL certificate](https://letsencrypt.org/){:target="_blank"}.
