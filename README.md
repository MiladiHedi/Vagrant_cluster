Vagrant Cluster
=============

This project provide a cluster of virtual machines ready to use for demos or POCs.
The vagrantFile will create 1 server master and two groups of machines in order to simulate 2 environments. It can be useful for an Ansible poc (by the way, in this code, ansible is installed on the server master).

##### Master server :
The master node is useful for Ansible but in many other cases, as in a real environment to handle a cluster, we connect to a machine which for security is the only one that can connect or control a cluster, and also in the case of our POC if the host is windows, it allows us to have a linux master server.
But you are free to remove the master or to put several master and change the topology if you wish.

##### Clusters :
All machines in the dev group can connect to each other using ssh.
All the machines of the prod group can connect to each other in ssh.
The master server can connect to all machines in ssh.
In the code you can see that there is a key pair which is used for all "dev" nodes, and another for the "prod" node.
This is an aberration, a private key must be owned by a single server, so in terms of security its forbidden, but I did it for the simplicity of implementation (I created the keys before the creation of the servers), and because it is not important in my use case but I am aware of it and I must warn you, even if it is obvious.