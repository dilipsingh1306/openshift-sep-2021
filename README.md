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
docker run -dit --name ubuntu1 --hostname ubuntu2 ubuntu:20.04 /bin/bash
docker run -dit --name ubuntu2 --hostname ubuntu2 ubuntu:20.04 /bin/bash
```
The expected output is
<pre>

</pre>
