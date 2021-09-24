### Installing Helm Kubernetes Package Manager (RPS User)
```
cd ~/Downloads
wget https://get.helm.sh/helm-v3.7.0-linux-amd64.tar.gz
tar xvfz helm-v3.7.0-linux-amd64.tar.gz
cd linux-amd64
ls
mv helm /usr/bin/helm
```
The expected output is
<pre>
root@ubuntu:/home/jegan/Downloads# <b>tar xvfz helm-v3.7.0-linux-amd64.tar.gz</b>
linux-amd64/
linux-amd64/helm
linux-amd64/LICENSE
linux-amd64/README.md
root@ubuntu:/home/jegan/Downloads# cd linux-amd64/
root@ubuntu:/home/jegan/Downloads/linux-amd64# ls
helm  LICENSE  README.md
root@ubuntu:/home/jegan/Downloads/linux-amd64# mv helm /usr/bin/helm
</pre>

### Let's create a custom helm chart(package)
```
cd ~/Training/openshift-sep-2021
git pull
cd Day5/
helm create nginx
tree nginx
```
The expected output is
<pre>
root@ubuntu:~/openshift-sep-2021/Day5/helm-demo# tree nginx
nginx
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

### Let's create a helm chart for nginx deployment
YOu need to remove all the files and directories inside nginx/templates folder and copy your deployment manifest files(s).
```
cd ~/Training/openshift-sep-2021
cd Day4/helm-demo/nginx/templates
rm -rf *
cp ../../*.yml .
cd ../..
helm create nginx
```
<pre>
root@ubuntu:~/openshift-sep-2021/Day5/helm-demo# ls
nginx
root@ubuntu:~/openshift-sep-2021/Day5/helm-demo# helm package nginx
Successfully packaged chart and saved it to: /root/openshift-sep-2021/Day5/helm-demo/nginx-0.1.0.tgz
root@ubuntu:~/openshift-sep-2021/Day5/helm-demo# 
</pre>
