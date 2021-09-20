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
- Virtualization came around year 2000
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
       - Parallels (Mac - OSX)
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
