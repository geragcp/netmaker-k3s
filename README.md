# netmaker-k3s
Run netmaker on a k3s server with defaults in a single pod

I was looking at netmaker the other day and thought: wouldn't it be nice if it could be deployed more easily to kubernetes? 
The project offers a helm chart that adds high availability if you in need of such a thing. 

K3s shipps with traefik and the official docker-compose file uses caddy instead. The advantage here of using k3s over docker is that
a pod is a network namespace. Everything in a pod shares the same loopback interface. This feature allows to remove all the 
natting and iptables mangling that netmaker has to do in docker. 

This project contains a deployment that puts the netmaker backend, ui, mqtt and coredns into the same pod. The communication between these services
is then facilitated via the loopback interface. There is of course a drawback in security but at least netmaker is easy to run this way.
The database is a sqlite file for simplicity. All the persistant volumes are mapped to a local folder for easy access just like a docker volume mount.

I am using traefik with let'sencrypt but this can be swapped out easily with your choice of ingress. 

### 01-namespace.yaml
will create a namespace

### 02-pvc.yaml
Before applying this you need to create two folders. Once for dns data and one for the sqlite database. 
Fopr example: mkdir -p /srv/netmaker/dns and chmod 777 /srv/netmaker/dns. Then add it where it says <PATH>
But you can see that there are three pvc's. Well k3s loca-path does not support readWriteMany. Since the dns pvc is mounted to netmaker and coredns at the same time, the workaround is to simply make two pvc's to the same path on disk.

### 03-nodeports.yaml
These are nodeports for wireguard connections to the container. Every network needs it's own port. There is a comms (management) network that is automatically
created for you on first start by netmaker. This file creates six nodeports for you. That means you can create five networks before you have to go in here and just add more. 

### 04-mqtt-configmap.yaml
This just needs to be there. It's from the official docker netmaker config. 

### 05-ingress.yaml
You need three domains with let'sencrypt. The dashboard, api for the dashboard and grpc. Note here the special rgpc config. It's not http.
Replace NETMAKER_BASE_DOMAIN with your domain name. 
  
### 06-deployment.yaml
You need to replace NETMAKER_BASE_DOMAIN with your domain in all the spots. Don't forget the one under the ui container. 
  
### Tip
Once it is running, Coredns created the corefile in the path you specified in 02-pvc. Go there and edit the default forwarder if you like. 
It creates it with 8.8.8.8 but I like to send it to a pi-hole instead. Your choice. 
