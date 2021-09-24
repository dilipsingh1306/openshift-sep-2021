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
