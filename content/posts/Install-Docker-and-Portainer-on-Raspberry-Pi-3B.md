---
title: "Install Docker and Portainer on Raspberry Pi 3B"
date: 2021-05-12T16:26:25+05:30
draft: false
tags: [raspberry-pi,docker]
---



**Table of Contents**

1. [About Docker and Portainer](#about-docker-and-portainer)
2. [Instructions to install Docker and Portainer](#instructions-to-install-docker-and-portainer-on-raspberry-pi)
   1. [Docker installation](#docker-installation)
   2. [Portainer installation](#portainer-installation)
3. [Reference](#reference)

****



## About Docker and Portainer

Docker might be a familar term. If this term is new to you, then Docker is a **container management service**. Dockers are pretty lightweight and have the ability to reduce the size of an application through a smaller footprint of the operating system and the application infrastructure.

Docker containers can be deployed anywhere, on any physical machines,virtual machines or also on the cloud.

Portainer is an opensource container management tool that can be used with Kubernetes,Docker or Docker Swarm. It allows anyone to deploy and manage containers with a userfriendly interface.

****



## Instructions to install Docker and Portainer on Raspberry Pi

### Docker installation

1. Ensure the Raspberry Pi software is upto date by running the below commands    
   
   ```bash
   sudo apt update && sudo apt upgrade
   ```

2. When the software update is completed successfully,we can then install docker through the below command.
   
   ```bash
   curl -sSL https://get.docker.com | sh
   ```

3. When the script completes successfully we have to provide the pi account access to Docker.
   
   ```bash
   sudo usermod -aG docker pi
   ```

   *NOTE:  If you have configured a different user account replace pi in the above command with the account name*

### Portainer installation

1. When the user account has been added successfully , we will now download Portainer image from the Docker hub for ARM processor (which is suitable for Raspberry Pi)
   
   ```bash
   sudo docker pull portainer/portainer-ce:linux-arm
   ```

2. We will now create a new container to run Portainer and configure it to run on port 9000.
   
   If the port 9000 is already in use, please use a different port number in the below command.
   
   ```bash
   sudo docker run --restart always -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:linux-arm
   ```

        ![portainer-installation](/images/2021/03/install-start-portainer.png)      

3. Navigate to the IP address of the raspberry pi and port 9000 to access the Portainer admin console
   
   ```bash
   http://[RASPBERRY_PI-<IP-ADDRESS>]:9000
   ```

        ![portainer-admin-console](/images/2021/03/portainer-admin-console.png)

****



## Reference

- [Portainer](https://www.portainer.io/)
