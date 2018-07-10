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

You can run the playbool with this command :
```
$ ansible-playbook -i hosts playbooks/install-registry.yml --ask-become-pass --ask-pass
```

you can check on the master node if the registry is running :
```
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                            NAMES
33e26b597327        registry:2          "/entrypoint.sh /etc…"   About a minute ago   Up About a minute   0.0.0.0:443->443/tcp, 5000/tcp   registryhttps

$ curl -k https://172.20.10.2/v2/_catalog
{"repositories":[]}
```
