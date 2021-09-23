### Kubernetes API Reference
```
https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#-strong-api-groups-strong-
```

### Service
 - represents a group of pods of same type
 - acts as a load balancer
 - acts as an abstraction layer between the client and the backend pods
 - eg:- 
    - There is a Java microservice application 
      Has two main layers/tiers
      1. Frontend(Microservice)
      2. Backend (MongoDB, CouchBase, etc.,)

For further reference, you may read this
https://kubernetes.io/docs/concepts/services-networking/service/

### Types of Services supported by Kubernetes
- ClusterIP
- NodePort
- LoadBalancer

#### Cluster IP Service
 - It is an internal service

#### NodePort
 - It is an external service

### LoadBalancer
 - It is an external service targetted for Cloud environments

### Creating a NodePort service (External Service)
```
kubectl create deployment nginx --image=nginx:1.20
kubectl scale deploy nginx --replicas=4
kubectl expose deploy nginx --type=NodePort --port=80
kubectl describe svc/nginx
```
The expected output is
<pre>
[root@master openshift-sep-2021]# kubectl expose deploy nginx --type=NodePort --port=80
service/nginx exposed
[root@master openshift-sep-2021]# kubectl describe svc/nginx
Name:                     nginx
Namespace:                default
Labels:                   app=nginx
Annotations:              <none>
Selector:                 app=nginx
Type:                     <b>NodePort</b>
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.98.240.67
IPs:                      10.98.240.67
Port:                     <b>80/TCP</b>
TargetPort:               80/TCP
NodePort:                 <b>30491/TCP</b>
Endpoints:                192.168.189.81:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
</pre>


You can access the NodePort as shown below
```
curl http://master:30491
curl http://worker1:30491
curl http://worker3:30491
```

### Creating a Cluster service (Internal Service)
```
kubectl delete svc/nginx
kubectl expose deploy nginx --type=ClusterIP --port=80
kubectl describe svc/nginx
```
The expected output is
<pre>
[root@master openshift-sep-2021]# <b>kubectl describe svc/nginx</b>
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              <b>ClusterIP</b>
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                <b>10.99.150.182</b>
IPs:               10.99.150.182
Port:              <b>80/TCP</b>
TargetPort:        80/TCP
Endpoints:         192.168.189.81:80
Session Affinity:  None
Events:            <none>
</pre>

You can access the ClusterIP Service as shown below
```
curl http:10.99.150.182//:80
```
Optionally, you can also access the ClusterIP by its name(service discovery)
```
kubectl exec -it nginx-depl-5fd5bfb4cf-29v62 sh
curl http://nginx:80
```
<pre>
[root@master openshift-sep-2021]# <b>kubectl exec -it nginx-depl-5fd5bfb4cf-29v62 sh</b>
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# <b>curl http://nginx:80</b>
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
</pre>


### Creating a LoadBalancer service (Cloud External Service)
Ideally you should only create LoadBalancer service in AWS, GCP, Azure, etc.,
When you attempt to create LoadBalancer Service in AWS,GCP or Azure, it will end up creating an ALB/NLB.
The Application Load Balancer or Network Load Balancer will load balance your pods running on different K8s worker nodes.

Since we are trying this locally, don't expect any external Load Balancer created :)

Locally it works just like NodePort Service.
```
kubectl delete svc/nginx
kubectl expose deploy nginx --type=LoadBalancer --port=80
kubectl describe svc/nginx
```
The expected output is
<pre>
[root@master openshift-sep-2021]# <b>kubectl describe svc/nginx</b>
Name:                     nginx
Namespace:                default
Labels:                   app=nginx
Annotations:              <none>
Selector:                 app=nginx
Type:                     <b>LoadBalancer</b>
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.97.194.244
IPs:                  30200    10.97.194.244
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <b>31154/TCP</b>
Endpoints:                192.168.189.81:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
</pre>

You may access the LoadBalancer locally as
```
curl http://master:31154
curl http://worker1:31154
curl http://worker2:31154
```

### Installing Nginx based Ingress Controller on our 3 Node K8s Cluster

#### Install ingress controller (Master Node only)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.1/deploy/static/provider/baremetal/deploy.yaml
```

#### Verify the installation
```
kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --watch
```

#### Finding the version of Ingress Controller installed
```
POD_NAMESPACE=ingress-nginx
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx --field-selector=status.phase=Running -o jsonpath='{.items[0].metadata.name}')

kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
```
The expected output is
<pre>
[root@master openshift-sep-2021]# <b>kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version</b>
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v1.0.1
  Build:         abab0396757dcd6f72018ee66611db18df838b17
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.9
-------------------------------------------------------------------------------
</pre>

### Checking if your Nginx Ingress Controller is ready to serve you
```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```
The expected output is
<pre>
[root@master openshift-sep-2021]# kubectl wait --namespace ingress-nginx \
>   --for=condition=ready pod \
>   --selector=app.kubernetes.io/component=controller \
>   --timeout=120s
<b>pod/ingress-nginx-controller-db898f6c7-ppbmt condition met</b>
</pre>
