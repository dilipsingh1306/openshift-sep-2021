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
</pre>

### Let's create a helm chart for nginx deployment
YOu need to remove all the files and directories inside nginx/templates folder and copy your deployment manifest files(s).

```
cd ~/Training/openshift-sep-2021
cd Day4/helm-demo/nginx/templates
rm -rf *
cp ../../*.yml .
cd ../..
<b>helm package nginx</b>
```

The expected output is

<pre>
root@ubuntu:~/openshift-sep-2021/Day5/helm-demo# ls
nginx
root@ubuntu:~/openshift-sep-2021/Day5/helm-demo# helm package nginx
<b>Successfully packaged chart and saved it to: /root/openshift-sep-2021/Day5/helm-demo/nginx-0.1.0.tgz</b>
root@ubuntu:~/openshift-sep-2021/Day5/helm-demo# 
</pre>

### Installing nginx using the nginx helm chart that we created in the previous step
```
cd ~/Training/openshift-sep-2021
git pull
cd Day5/helm-demo
helm install nginx nginx-0.1.0.tgz
```
The expected output is
<pre>
root@master helm-demo]# ls
nginx  nginx-0.1.0.tgz
[root@master helm-demo]# <b>kubectl get deploy</b>
No resources found in default namespace.
[root@master helm-demo]# <b>helm install nginx nginx-0.1.0.tgz
NAME: nginx
LAST DEPLOYED: Fri Sep 24 00:21:38 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
</b>
[root@master helm-demo]# <b>kubectl get deploy</b>
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   4/4     4            4           22s
[root@master helm-demo]# kubectl get po
NAME                               READY   STATUS        RESTARTS       AGE
nginx-7fb9867-54jrm                1/1     Running       0              45s
nginx-7fb9867-thds2                1/1     Running       0              45s
nginx-7fb9867-vvwd8                1/1     Running       0              45s
nginx-7fb9867-ztqb7                1/1     Running       0              45s
</pre>
