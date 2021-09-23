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
Assuming the NodePort assigned to nginx service is 30200, you can access the NodePort as shown below
```
curl http://master:30200
curl http://worker1:30200
curl http://worker3:30200
```

### Creating a Cluster service (Internal Service)
```
kubectl delete svc/nginx
kubectl expose deploy nginx --type=NodePort --port=80
kubectl describe svc/nginx
```
The expected output is
<pre>
[root@master openshift-sep-2021]# kubectl describe svc/nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.99.150.182
IPs:               10.99.150.182
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         192.168.189.81:80
Session Affinity:  None
Events:            <none>
</pre>

Assuming the NodePort assigned to nginx service is 30200, you can access the NodePort as shown below
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
