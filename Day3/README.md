### Storing Docker Login Credentials as Secrets in Kubernetes

First we need to perform a docker login 
```
su -
docker login
```
When it prompts for your docker user and password, type the same.  Your docker credential will be stored
in /root/.docker/config.json

We need to create Kubernetes secrets to store the docker credentials with K8s cluster securely. This secret will then
be used to pull the Docker images.

```
kubectl create secret generic regcred \
   --from-file=.dockerconfigjson=/root/.docker/config.json \
   --type=kubernetes.io/dockerconfigjson
```

You may verify if the secret is created
```
kubectl get secrets
```
The expected output is
<pre>
[root@master ~]# <b>kubectl create secret generic regcred \
> --from-file=.dockerconfigjson=/root/.docker/config.json \
> --type=kubernetes.io/dockerconfigjson</b>
secret/regcred created
[root@master ~]# kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-sft5f   kubernetes.io/service-account-token   3      135m
<b>regcred               kubernetes.io/dockerconfigjson        1      8s</b>
</pre>

