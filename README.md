# openshift-sep-2021

### Boot Loaders
- LILO (Linux Loader)
- GRUB1
- GRUB2
- you can install multiple OS in the same PC/Workstation/Server but only one OS can be active(booted) while OS are passive(shutdown)
- The number of OS that can be installed depends on max number of primary partitions(4)

### Hypervisor

### What is Hypervisor?

- nothing but Virtualization Technology
- Virtualization came around year 2000tektutorjegan
- Virtualization is a technology that helps boot mulitple OS simultaneously on the same PC/Workstation/Server
- Is it software Technology?
- Is it a Hardware Technology?
- is it combination of Software & Hardware?
   - Virtualization Softwares (VMWare, Oracle, Parallels, Hyper-V, etc.,)
       - VMWare
           - VMWare Workstation 
           - VMWare Fusion (Mac OSX) - Type 1 Hypervisor (requires Host OS)
           - VMWare Player (Windows) - Type 1 Hypervisor (requires Host OS)
           - VMWare vSphere (Bare Metal Hypervisor - Type 2) - Doesn't require any OS
               - Meant to be used in Workstations & Servers
               - Primary of Servers/Workstations maybe to host multiple Virtual Machines
       - Oracle 
           - VirtualBox (Free for Commercial use as well supported on Unix/Linux/Widows & Mac) 
       - Parallels (Mac - OSX)docker run --dit --name ubuntu1 --hostname ubuntu2 ubuntu:20.04 /bin/bash

       - Microsoft
           - Hyper-V
- heavy-weight technology
    - Every OS that is installed on top of Hypervisor is referred as Guest OS(Virtual Machine)
    - Each Virtual Machines 
        - requires dedicated hardware resources
        - eg. CPU, RAM, Storage, etc.,
    - The maximum number of Virtual Machines that can hosted by a single Workstation/Server/Desktop/Laptop depends on
      primarily the Processor(the number of CPU cores it supports)

   - Hardware
      - BIOS Firmware also need to support Virtualization
      - Processors
          - AMD Processor
              - AMD-V is the virtualization CPU Feature name supported by AMD
          - Intel Processor
              - VT-X is the virtualization CPU Feature name supported by Intel 
 
 Container Technology
   - lightweight technology
       - hardwares resources availbale on the machine where containers are running are shared by all containers
       - no dedicated hardwares resources are required for each container
       - containers are application process not Operating System or not Virtual Machine(completely functional OS)
       - each containers runs in its own namespace(
   - 
   - each container has the below
       - has an IP Address
       - has shell (bash, korn, sh, etc)
       - file system
       - host name
       - Network Stack
   - is a Linux Technology
   - does not have their own Kernel.  Kernel is shared
   - Linux Kernel supports 2 major features
       1. Namespace (Isolation)
       2. CGroups ( Control Groups - resource quota restrications can be applied for each containers )
   - Recommendation
      - one application per container
      - you may run multiple applications per container with some additional overhead
           - supervisord monitoring tool this will create additional child processes to support addition applications
   
   - different container runtimes available
       - LXC (Light Weight Containership )
       - Docker - has a Community Edition(Free) and a Enterprise Edition(Commercial use - License required)
       - Podman - 1-to-1 compatiable with Docker commands, 
       - CRI-O
   
   
   RHEL, Fedora, CentOS, Ubuntu, SUSE, etc.,
      - Different Linux Distributions
         - Each distribution may distribute a particular
              - version of Linux kernel
              - version of Shell ( Bash, Sh, etc,)
              - version of Linux tools ( vim, ls, cp, scp, sftp, rm, grep )
      - Common - Linux OS
          - they all use the same Linux Kernel


### Local Docker Registrydocker pull hello-world:latest

  - /var/lib/docker is the folder where the Docker images, image layers, container, etc are mainted

### Private Docker Registry
  - can be setup using registry:2 docker image ( good for learning, not recommended for production )
  - can also setup using Sonatype Nexus or JFrog Artifactory ( Recommended for Production )
  - can host Proprierary Docker Images within your organization giving access to restricted users

### Docker Hub (Dock Hub website mainted by Docker Inc)
  - Public Docker Remote Repostiroy
  - There is a community maintained Repository for Open Source Docker-CE and separate repository for Docker-EE.
  - Has a wide range of open source Docker Images

### Docker Image
  - a blueprint of a Docker container
  - specification of a Docker containers
  - any tools installed in the Docker image are available on the container
  - images are used to create a docker container
  - every images is allocated a unique name and ID

### Docker Container
  - is a instance of a Docker Image
  - with a single docker image, any number of Docker containers can be created
  - every container is alloted with a Private IP by default
  - every container gets a unique name and ID

### Installing Docker-CE in CentOS
```
su -
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce --allowerasing
systemctl enable docker && sudo systemctl start docker
docker --version
docker images
```
When prompted for root password, type rps@12345

### Try docker commands as non-root user(rps)
```
docker images
```
You may get the below error
<pre>
[jegan@tektutor ~]$ docker images
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/json": dial unix /var/run/docker.sock: connect: permission denied
</pre>

As the 'jegan' user, in your case 'rps' user isn't part of docker user group, the rps user isn't able to write to
the socket.

Login as root(administrator) user
```
su -
whoami
```
Type your root password which is rps@12345
The expected output is
<pre>
[jegan@tektutor ~]$ su -
Password: 
Last login: Mon Sep 20 02:01:21 PDT 2021 on pts/0
[root@tektutor ~]# whoami
root
</pre>

Add the rps user to the docker user group
```
usermod -aG docker rps
exit
sudo su rps
docker images
```
The expected output is
<pre>
[jegan@tektutor ~]$ sudo su jegan
[sudo] password for jegan: 
[jegan@tektutor ~]$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
</pre>

### Listing docker images from Local Docker Registry
```
docker images
```

### Downloading a Docker image from Docker Hub website(Docker Remote Registry) to Local Registry
```
docker pull hello-world:latest
```

### Creating containers in background(daemon/deattached) mode
```
docker run -dit --name ubuntu1 --hostname ubuntu1 ubuntu:20.04 /bin/bash
docker run -dit --name ubuntu2 --hostname ubuntu2 ubuntu:20.04 /bin/bash
```
The expected output is
<pre>
[jegan@tektutor ~]$ <b>docker run -dit --name ubuntu1 --hostname ubuntu1 ubuntu:20.04 /bin/bash</b>
Unable to find image 'ubuntu:20.04' locally
20.04: Pulling from library/ubuntu
35807b77a593: Pull complete 
Digest: sha256:9d6a8699fb5c9c39cf08a0871bd6219f0400981c570894cd8cbea30d3424a31f
Status: Downloaded newer image for ubuntu:20.04
e5c04224c7488c9a4b15b9bf3c9de78e6f8e52042b562984c9a295ece6e72759
[jegan@tektutor ~]$ <b>docker run -dit --name ubuntu2 --hostname ubuntu2 ubuntu:20.04 /bin/bash</b>
417b2faf758be6ea3b690701f964d6d8bdcc0f832e00238cd7f2e5bcf86ff78c

[jegan@tektutor ~]$ docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS     NAMES
417b2faf758b   ubuntu:20.04   "/bin/bash"   2 seconds ago    Up 1 second               ubuntu2
e5c04224c748   ubuntu:20.04   "/bin/bash"   24 seconds ago   Up 23 seconds             ubuntu1
</pre>

### Getting inside the container shell
```
docker exec -it ubuntu1 /bin/bash
```
The expected output is
<pre>
[jegan@tektutor ~]$ docker exec -it ubuntu1 /bin/bash
root@ubuntu1:/# hostname
ubuntu1
root@ubuntu1:/# hostname -i
172.17.0.2
root@ubuntu1:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@ubuntu1:/# <b>exit</b> 
</pre>

### Listing all currently running containers
```
docker ps
```

### Listing all containers ( containers that are merely created, running and exited, etc.,)
```
docker ps -a
```

### Stopping a running container
```
docker stop ubuntu1
```

### Stopping multiple containers
```
docker stop ubuntu2 ubuntu3 ubuntu4
docker stop $(docker ps -q)
```

### Starting a exited container
```
docker start ubuntu1
```

### Starting multiple containers
```
docker start ubuntu2 ubuntu3 ubuntu4
docker start $(docker ps -aq)
```

### Restarting a container
```
docker restart ubuntu1
```

### Restarting multiple containers
```
docker restart ubuntu1 ubuntu2 ubuntu3 ubuntu4
docker restart $(docker ps -q)
```

### Delete a single container graciously
```
docker stop ubuntu1
docker rm ubuntu1
```

### Deleting multiple containers graciously
```
docker stop $(docker ps -q) && docker rm $(docker ps -aq)
```

### Deleting multiple containers forcibly
```
docker rm -f $(docker ps -aq)
```

### Renaming a container
```
docker rename ubuntu1 server1
```
<pre>
ubuntu1 - current container name
server1 - new container name
</pre>
You may check the new name 
```
docker ps
```

### Finding the IP address of a container
```
docker inspect ubuntu1 | grep IPA
docker inspect -f {{.NetworkSettings.IPAddress}} ubuntu1
```

### Creating a custom docker network
Let's create a folder in /home/rps directory.
```
cd /home/rps
mkdir Training
cd Training
```
Create a file named Dockerfile with the below content in the /home/rps/Training folder.
```
FROM ubuntu:20.04
MAINTAINER Jeganathan Swaminathan <jegan@tektutor.org>

RUN apt-get update && apt-get install -y net-tools iputils-ping
```
Let us now build the custom image
```
cd /home/rps/Training
docker build -t tektutor/ubuntu-net-tools:latest .
```

### Creating custom network in Docker
```
docker network create net-1
docker network create net-2
```

### Listing the networks
```
docker network ls
```

### Inspecting network details
```
docker network inspect net-1
```

### Inspecting network details
```
docker network inspect net-2
```

### Creating new containers attaching it to our custom network
```
docker run -dit --name c1 --hostname c1 --network net-1 tektutor/ubuntu-net-tools:latest /bin/bash
docker run -dit --name c1 --hostname c1 --network net-1 tektutor/ubuntu-net-tools:latest /bin/bash
```

### Listing the container whose name starts with letter c
```
docker ps --filter "--name=c*" 
```

### Finding IP details of c1 and c2
```
docker inspect -f {{.NetworkSettings.IPAddress}} c1
docker inspect c2 | grep IPA
```

### Getting inside c1
```
docker exec -it c1 bash
ping 172.19.0.2
exit
```
As you can observe c1 isn't able to ping c2 container(IP - 172.19.0.2).

### Getting inside c2
```
docker exec -it c2 bash
ping 172.18.0.2
exit
```
As you can observe c2 isn't able to ping c1 container(IP - 172.18.0.2).

### Let us now connect c1 container to net-2 in addition to already connect net-1 network
```
docker network connect net-2 c1
```

### Inspect container c1 to observe two IP addresses for c1 container
```
docker inspect c1 | grep IPA
ping 192.19.0.2
exit
```
Now c1 should be able to ping c2.

### Now ping c2 from c1
```
docker exec -it c2 | grep IPA
ping 192.19.0.3
exit
```
Now c2 should be able to ping c1.
