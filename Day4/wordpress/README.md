### Installing Git
```
su -
yum install -y git
```

### Enabling storage and dns addons in microk8s
```
su -
microk8s enable storage
microk8s enable dns
```

### Deploying wordpress (perform this as root user)
```
su -
mkdir Training
git clone https://github.com/tektutor/openshift-sep-2021.git
cd openshift-sep-2021/Day4/manifests
microk8s kubectl apply -k ./
```
### Installing Helm Kubernetes Package Manager (RPS User)
```
cd ~/Downloads
wget https://get.helm.sh/helm-v3.7.0-linux-amd64.tar.gz
tar xvfz helm-v3.7.0-linux-amd64.tar.gz
cd linux-amd64
```
The expected output is
<pre>
root@ubuntu:/home/jegan/Downloads# <b>tar xvfz helm-v3.7.0-linux-amd64.tar.gz</b>
linux-amd64/
linux-amd64/helm
linux-amd64/LICENSE
linux-amd64/README.md
</pre>
