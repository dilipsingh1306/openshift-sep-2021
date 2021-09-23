### Setting up a load balancer using containers
We need to do port forwarding only for lb container as lb is the only container which is outward facing.
While web1 to web5 are just going to be maintained locally visible with private IPs with no port-forwarding, 
hence no one can access web1 to web5 webserver from outside world, which is secured.

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
The assumption is
<pre>
172.17.0.3 - IP Address of web1
172.17.0.4 - IP Address of web2
172.17.0.5 - IP Address of web3
172.17.0.6 - IP Address of web4
172.17.0.7 - IP Address of web5
</pre>

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
docker cp web1:/usr/share/nginx/html/index.html
```

### Modifying the web2 web page
```
echo "Web Server 2" > index.html
docker cp web1:/usr/share/nginx/html/index.html
```

### Modifying the web3 web page
```
echo "Web Server 3" > index.html
docker cp web1:/usr/share/nginx/html/index.html
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

### Volume Mounting
Let us create a mysql container
```
docker run -d --name mysql1 --hostname mysql1 -e MYSQL_ROOT_PASSWORD=root mysql:8
```

Let us list and see if mysql1 container is running
```
docker ps
```

Let us get inside the mysql1 container
```
docker exec -it mysql1 sh
mysql -u root -p 
CREATE DATABASE tektutor;
USE tektutor;
CREATE TABLE Training VALUES (id int, name VARCHAR(25), duration VARCHAR(25));
INSERT INTO Training VALUES ( 1, "Mastering C++", "5 Days" );
INSERT INTO Training VALUES ( 2, "Advanced Python", "5 Days" );
INSERT INTO Training VALUES ( 3, "DevOps", "3 Days" );
SELECT * FROM Training;
exit
exit
```
Let us mysql1 container, along with the container your data has gone.
```
docker rm -f mysql1
```

With volume mounting, we will not loose data when the container is delete/aborted.
```
docker run -d --name mysql1 --hostname mysql1 -e MYSQL_ROOT_PASSWORD=root -v /tmp:/var/lib/mysql mysql:8

docker exec -it mysql1 sh

mysql -u root -p 
CREATE DATABASE tektutor;
USE tektutor;
CREATE TABLE Training VALUES (id int, name VARCHAR(25), duration VARCHAR(25));
INSERT INTO Training VALUES ( 1, "Mastering C++", "5 Days" );
INSERT INTO Training VALUES ( 2, "Advanced Python", "5 Days" );
INSERT INTO Training VALUES ( 3, "DevOps", "3 Days" );
SELECT * FROM Training;

exit
exit
```
Now let's remove the mysql1 container. Even though we removed mysql1 container your data is intact.
```
docker rm -f mysql1
```

Let's create a new mysql2 container mounting the same path
```
docker run -d --name mysql2 --hostname mysql2 -e MYSQL_ROOT_PASSWORD=root -v /tmp:/var/lib/mysql mysql:8

docker exec -it mysql2 sh

mysql -u root -p 
SHOW  DATABASES;
USE tektutor;
SHOW TABLES;
SELECT * FROM Training;
```
The expected output is
<pre>
[jegan@tektutor /]$ docker exec -it mysql2 sh
# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.26 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| tektutor           |
+--------------------+
5 rows in set (0.01 sec)

mysql> USE tektutor;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT * FROM Training;
+------+----------------------+----------+
| id   | name                 | duration |
+------+----------------------+----------+
|    1 | DevOps               | 5 Days   |
|    2 | Qt & QML Programming | 5 Days   |
|    3 | Advanced Scala       | 5 Days   |
+------+----------------------+----------+
3 rows in set (0.00 sec)

mysql> 

</pre>

With this lab exercise, we would understand that our data is safe even though mysql1 container was removed.  
We could still access the data created within mysql1 container via mysql2 container because the data is stored
externally in the host path /tmp.

For production, using hostpath isn't recommended.  Hence you may consider using AWS S3 or similar storage services as 
Volume.

### Installing MicroK8s Kubernetes in Worker Node2

#### We need to first install snap package manager in CentOS 8.x
The below commands can be execute as a whole.
```
su -
yum install -y epel-release
yum install -y snapd
systemctl enable --now snapd.socket
ln -s /var/lib/snapd/snap /snap
```

#### Now let's install Microk8s cluster
You need to execute the below commands one at a time.
```
su - 
snap install microk8s --classic 
microk8s status --wait-ready
microk8s kubectl get all --all-namespaces 
echo 'export PATH=$PATH:/var/lib/snapd/snap/bin' | sudo tee -a /etc/profile.d/mysnap.sh
```
Close you terminal, launch the terminal and login as root user

Adding microk8s in path permanently
```
vim ~/.bashrc
```
And append the below line at as the end of the file
```
export PATH=/var/lib/snapd/snap/bin:$PATH
```
To apply the .bashrc changes immediately, you need to run
```
source ~/.bashrc
```

You may test if your K8s cluster is up and running by trying the below command
```
microk8s kubectl get nodes
```
The expected output is
<pre>
[root@tektutor ~]# microk8s kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
tektutor   Ready    <none>   10m   v1.21.4-3+e5758f73ed2a04
</pre>

### Creating your first deploy in Kubernetes (microk8s)
```
microk8s kubectl create deploy nginx --image=nginx:1.20
```

### Listing all deployments
```
microk8s kubectl get deployments
microk8s kubectl get deploy
```

### List all replicasets
```
microk8s kubectl get replicasets
microk8s kubectl get replicaset
microk8s kubectl get rs
```

### List all pods
```
microks8 kubectl get pods
microk8s kubectl get pod
microk8s kubectl get po
```
