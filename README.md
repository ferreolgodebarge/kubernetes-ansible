# Project

I will install with Ansible a very simple kubernetes on 2 VMs locally in order to test the solution and its deployment.

On the master node :
- kube-scheduler (K8s)
- kube-controller (K8s)
- api-kube (K8s)
- Ansible (Automation)
- Docker-CE (Container Engine)
- Registry V2 (Registry)

On the worker node :
- kubelet (K8s)
- Docker-CE (Container Engine)

# System configuration

I have 2 VMs built with Virtualbox :

```
$ ls_release -a
Distributor ID: Ubuntu
Description:    Ubuntu 16.04.4 LTS
Release:        16.04
Codename:       xenial
```

IP addresses : 
- VM master : 172.20.10.2
- VM worker : 172.20.10.3

# Install Ansible

```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```

# My working directory

This is my directory tree :

```
|---kubernetes-ansible/
|------playbooks/
|---------install-docker.yml
|---------install-registry.yml
|---------install-etcd.yml
|------certs/
|---------domain.crt
|---------domain.key
|------hosts
```

The file hosts :
```
[master]
172.20.10.2

[worker]
172.20.10.3
```

This file is the inventory of reachable machines by Ansible

Note that Ansible is intalled on the master.

Command to test the connection to the nodes with ansible :
```
$ ansible -i hosts all -m ping --ask-pass
```

# Install Docker on all nodes

You can run the playbook with this command :
```
$ ansible-playbook -i hosts playbooks/install-docker.yml --ask-become-pass --ask-pass
``` 

you can check on the nodes if docker is installed :
```
$ sudo docker ps
```

# Install Docker registry on master node

First, we want to create self-signed certificates with openssl :
```
 openssl req -newkey rsa:2048 -nodes -keyout domain.key -x509 -days 365 -out domain.crt
```
You'll obtain a domain.crt and a domain.key files.

You can run the playbook with this command :
```
$ ansible-playbook -i hosts playbooks/install-registry.yml --ask-become-pass --ask-pass
```

you can check on the master node if the registry is running :
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                            NAMES
33e26b597327        registry:2          "/entrypoint.sh /etc…"   About a minute ago   Up About a minute   0.0.0.0:443->443/tcp, 5000/tcp   registryhttps

$ curl -k https://172.20.10.2/v2/_catalog
{"repositories":[]}
```

# Install ETCD on master node

We will take the Careos public image of ETCD.

This is the command to pull and run the ETCD image thanks to a playbook ansible :
```
ansible-playbook playbooks/install-etcd.yml --ask-become-pass --ask-pass -i hosts
```

In order to check if the etcd is running, you can try :
```
$ docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
c2fa4716398a        quay.io/coreos/etcd:v2.3.8   "/etcd -name etcd0 -…"   2 minutes ago       Up 2 minutes        0.0.0.0:2379-2380->2379-2380/tcp, 0.0.0.0:4001->4001/tcp, 7001/tcp   etcd
33e26b597327        registry:2                   "/entrypoint.sh /etc…"   2 hours ago         Up 2 hours          0.0.0.0:443->443/tcp, 5000/tcp                                       registryhttps

$ curl http://172.20.10.2:2379/version
{"etcdserver":"2.3.8","etcdcluster":"2.3.0"}
```

# Install Kubernetes cluster

In order to create a Kubernetes cluster, you can use a docker image called hyperkube.

### Pull Hyperkube image in the registry

You can pull this image :
```
docker pull googlecontainer/hyperkube
```
Then you will run this image in order to deploy : 
- kubelet
- api-server
- kube-scheduler
- kube-controller

### Install API Server


