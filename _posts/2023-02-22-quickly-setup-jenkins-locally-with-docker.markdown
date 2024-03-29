---
layout: "post"
title:  "Quickly setup jenkins locally with docker"
date:   "2023-02-22 10:30:00 -0500"
categories: docker, jenkins
---

This is a quick guide to get jenkins up and running locally in few mins.


####  Prerequisite
You need to have Docker installed on your machine, here's link to the [official documentation](https://docs.docker.com/get-docker/).


#### Step 1. Create volume

If you'd like to persist jenkins data when the container is restarted, you need to create a docker volume.
```bash
docker volume create [your_host_volume]
```

You can inspect the volume which the follow command:
```bash
docker inspect [your_host_volume]
```

#### Step 1. Start jenkins
```bash
docker container run -d \
    -p [your_host_port]:8080 \
    -v [your_host_volume]:/var/jenkins_home \
    --name jenkins-local \
    jenkins/jenkins:lts
```

`-p` binds 8080 of the container port to your host port.

`-v` mounts /var/jenkins_home in the container to your host volume created earlier.

<figure>
  <img src="{{site.url}}/img/jenkins.webp" alt="Jenkins"/>
</figure>

#### Step 1. Retrieve jenkins password
As part of initial setup, you need to retrieve the password from the container path below.

```bash
docker container exec \
    [container_id] \
    sh -c "cat /var/jenkins_home/secrets/initialAdminPassword"
```

Jenkins should be available at `localhost:your_host_port`.