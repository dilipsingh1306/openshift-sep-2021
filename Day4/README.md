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
