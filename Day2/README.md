### Setting up a load balancer using containers
```
docker run -d --name lb --hostname lb -p 80:80 nginx:1.20
docker run -d --name web1 --hostname web1 nginx:1.20
docker run -d --name web2 --hostname web2 nginx:1.20
docker run -d --name web3 --hostname web3 nginx:1.20
docker run -d --name web4 --hostname web4 nginx:1.20
docker run -d --name web5 --hostname web5 nginx:1.20
```

### Configuring lb container as a Load Balancer
You may copy the nginx.conf file from lb to local machine and edit as shown below
```
docker cp lb:/etc/nginx/nginx.conf .
```

Now we need to edit the nginx.conf file as shown below

```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    upstream backend {
        server 172.17.0.3:80;
        server 172.17.0.4:80;
        server 172.17.0.5:80;
        server 172.17.0.6:80;
        server 172.17.0.7:80;
    }

    server {
       location / {
        proxy_pass http://backend;
       }
    }
}
```

#### Copy the nginx.conf file to lb container
```
docker cp nginx.conf lb:/etc/nginx/nginx.conf
docker restart lb
```
After coying the nginx.conf file into the lb, we need to restart the lb container to apply the config changes.

Check if the lb container is still running
```
docker ps
```

In case the lb container isn't running, it is possible nginx.conf file has some typos. Fixing nginx.conf and
copying and restart should fix the issue.

#### Modifying the web1 web page
```
echo "Web Server 1" > index.html
docker cp web1:/usr/share/html/index.html
```

### Modifying the web2 web page
```
echo "Web Server 2" > index.html
docker cp web1:/usr/share/html/index.html
```

### Modifying the web3 web page
```
echo "Web Server 3" > index.html
docker cp web1:/usr/share/html/index.html
```

Repeat the above process for web4 and web5.

### Testing if Load Balancer container is working as expected
Find the IP Address of your lab machine
```
ifconfig ens192
```

Check if you are able to access the web page from RPS CentOS Lab machine
```
curl localhost
curl localhost
curl localhost
```

Each time you do 'curl localhost' you are supposed to see responses coming from web1, web2, web3 in a round-robin fashion.

Hope you now understand how port-forwarding in Docker helps you access the container services from external machines. Also you by now you
would be able to appreciate, how a simple High Available(HA) website works with by performing this lab exercise. 


###  Testing High Availability
You may try stopping some containers and see what happens when try accessing the web page.
```
docker stop web4
```
