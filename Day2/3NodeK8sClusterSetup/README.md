# Installing your 3-Node Kubernetes Cluster v1.22 in OnPrem Servers(CentOS 8.3+) 

### You make like these articles, give it a try
<pre>
https://github.com/flannel-io/flannel
https://docs.openshift.com/container-platform/3.4/architecture/additional_concepts/flannel.html
https://www.tigera.io/blog/kubernetes-networking-with-calico/
https://docs.projectcalico.org/reference/architecture/overview
</pre>

### Disable Virtual Memory (swap parition) in Master and Worker Nodes
```
sudo swapoff -a
```
To permanently disable swap partition,  edit  the /etc/fstab file root user and comment the swap partition.
```
sudo vim /etc/fstab
sudo systemctl daemon-reload
```

### Disable SELINUX in Master and Worker Nodes
The below command disables selinux for now. SELinux will be disabled until you reboot your CentOS box.
``` 
setenforce 0
```

To permanently disable SELINUX, you need to edit /etc/selinux/config file and change enforcing to disabled.
```
sudo systemctl daemon-reload
```

### Configure the hostnames K8s nodes
#### In Master Node
```
sudo hostnamectl set-hostname master
```

#### In worker1 Node
```
sudo hostnamectl  set-hostname worker1
```

#### In worker2 Node
```
sudo hostnamectl set-hostnamme worker2
```

### Update /etc/hosts in Master and all worker nodes
This helps K8s nodes to resolve each other by the hostnames. K8s nodes will be named using the respective hostnames. 

#### Find the IP Address of your master node
My default Network Interface Card(NIC) name is `ens33'.  You may have to find your default NIC name.

```
ifconfig ens33
```
Note down the IP of master node as we need to add this in the /etc/hosts shortly.

### Find the IP Address of your worker1 node
```
ifconfig ens33
```
Note down the IP of worker1 node as we need to add this in the /etc/hosts shortly.

### Find the IP Address of your worker2 node
```
ifconfig ens33
```
Note down the IP of worker2 node as we need to add this in the /etc/hosts shortly.

### Configure /etc/hosts file ( Applicable for Master & Node Worker Nodes )
Append the IP Addresses of master, worker1 and worker2 as shown below in /etc/hosts files. 

```
192.168.254.129 master 
192.168.254.130 worker1
192.168.254.131 worker2
```

### Firewall configurations
For summary of ports that must be opened, refer official Kubernetes documentation page https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

#### Open the below ports in Master Node as root user
```
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=2379-2380/tcp
firewall-cmd --permanent --add-port=10250-10252/tcp
firewall-cmd --permanent --add-port=10254/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd --permanent --add-port=176-178/tcp
firewall-cmd --permanent --add-port=8285/tcp
firewall-cmd --permanent --add-port=8285/udp
firewall-cmd --permanent --add-port=8443/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp

firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --zone=trusted  --add-source=192.168.0.0/16 
firewall-cmd --permanent --zone=trusted  --add-source=172.16.95.0/24 
modprobe br_netfilter
systemctl daemon-reload
systemctl restart firewalld
systemctl status firewalld
firewall-cmd --list-all
```
You may have to open additional ports, depending on which CNI plugin you intend to use (Calico, Flannel, Weave etc.,)

In the above trusted zone, the IP 192.168.0.0/16 is the pod network cidr we intend to give while bootstrapping master node with Calico CNI. In case you prefer using Flannel CNI, you may need to replace 192.168.0.0/16 with 10.244.0.0/16.

172.16.95.0/24 is my Virtual Machine Subnet range.  You may find your Virtual Machine IPs and modify this accordingly.

#### Open the below ports in Worker Nodes as root user
```
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10254/tcp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --permanent --add-port=176-178/tcp
firewall-cmd --permanent --add-port=8285/tcp
firewall-cmd --permanent --add-port=8285/udp

firewall-cmd --permanent --add-port=8443/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp


firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --zone=trusted  --add-source=192.168.0.0/16
firewall-cmd --permanent --zone=trusted  --add-source=172.16.95.0/24 
modprobe br_netfilter
systemctl daemon-reload
systemctl restart firewalld
systemctl status firewalld
firewall-cmd --list-all
```

#### Install Docker CE in Master and Worker Nodes
```
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce --allowerasing
sudo usermod -aG docker jegan
sudo su jegan
```
You may have to replace jegan(non-admin user) with your non-admin user.

### Configure Docker Engine to use systemd driver in Master and Worker Nodes
You may not have /etc/docker folder by now, hence you need to issue the below command 
```
sudo systemctl enable docker && sudo systemctl start docker
```

sudo vim /etc/docker/daemon.json

```
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
     "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}

sudo mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo systemctl enable docker && sudo systemctl start docker
```

#### Configure IPTables to see bridge traffic in Master and Worker Nodes
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### Install kubectl kubeadm and kubelet on Master & Worker nodes
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

### Configure kubelet in Master and Worker Nodes
Edit /etc/sysconfig/kubelet and append the below line
```
KUBELET_EXTRA_ARGS=--runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice
```

You may enable the kubelet service as shown below
```
sudo systemctl enable --now kubelet
```

### Restart Docker and Kubelet before you bootstrap master
```
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```

### Bootstrapping Master Node as root user
```
su -
kubeadm init --pod-network-cidr=192.168.0.0/16
```
Once your `kubeadm init` is successfully completed, make sure you do the below
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

In order to access the cluster without issues after machine reboot, add the below to /root/.bashrc
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```
To force apply /root/.bashrc changes immediately
```
source /root/.bashrc
```
Also make sure you do this for your non-admin user terminal as well
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```
Export KUBECONFIG in your /home/jegan/.bashrc with below line appended at the end of the file.  Ofcourse, you need to replace 'jegan' with your non-admin user name :)
```
export KUBECONFIG=$HOME/.kube/config
```
To force apply changes done in bashrc
```
source ~/.bashrc
```

Save your join token in a file on the Master Node, the token varies on every system and every time you type kubeadm init it changes on the same system, hence you need to save your join token for your reference before you clear your terminal screen.

##### Create a file with name token or a better name and save your token.
```
kubeadm join 192.168.154.128:6443 --token 5zt7tp.2txcmgnuzmxtgnl \
        --discovery-token-ca-cert-hash sha256:27758d146627cfd92079935cbaff04cb1948da37c78b2beb2fc8b15c2a5adba
```

### Troubleshooting kubeadm init and join (Applicable for master and worker nodes)
```
su -
kubeadm reset
```

You need to manually remove the below folder
```
su -
rm -rf /etc/cni/net.d
rm -rf /etc/kubernetes
rm -rf $HOME/.kube
```
Now you may proceed with 'kubeadm init' with appropriate pod-network-cidr subnet.


#### In case you forgot to save your join token and cleared the terminal screen, no worries try this on Master Node
```
kubeadm token create --print-join-command
```

#### In Master Node
```
kubectl get nodes
kubectl get po -n kube-system -w
```
Press Ctrl+C to come out of watch mode.

#### Installing Calico CNI in Master Node
To learn more about Calico CNI, refer https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises 
```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

#### In case you wish to install Flannel as opposed to Calico
You can choose only one CNI at a time.  

Flannel expects you to bootstrap master node with 10.244.0.0/16 pod network cidr. In case you have used 192.168.0.0/16 or some other pod-network-cidr for a different CNI, you need to perform 'kubeadm reset' and 'kubeadm init' before you install Flannel CNI.

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### In Master Node watch the pod creation after installing Network CNI
```
kubectl get po -n kube-system -w
```
![Control Plane Pods](https://github.com/tektutor/kubernetes-may-2021/blob/master/Day2/3NodeClusterSetup/K8s-controlplane-pods.png)

Press Ctrl+C to come out of watch mode.

#### In Worker Node
I have given the join token here for your reference, you need to copy and paste your join token that is shown on your master bootstrapped terminal.

```
kubeadm join 192.168.154.128:6443 --token 5zt7tp.2txcmgnuzmxtgnl \
        --discovery-token-ca-cert-hash sha256:27758d146627cfd92079935cbaff04cb1948da37c78b2beb2fc8b15c2a5adba
```
![Worker Join Command](https://github.com/tektutor/kubernetes-may-2021/blob/master/Day2/3NodeClusterSetup/worker1-join.png)

#### In Master Node
At this point,  you are supposed to see 3 nodes in ready state.
```
kubectl get nodes
```
![Node List](https://github.com/tektutor/kubernetes-may-2021/blob/master/Day2/3NodeClusterSetup/node-list.png)

If you see similar output on your system, your 3 node Kubernetes cluster is all set !!!
