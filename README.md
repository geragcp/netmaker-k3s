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
