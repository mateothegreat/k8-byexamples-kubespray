# Rollin deep with Kubespray

This file file serves as your book's preface, a great place to describe your book's content and ideas.

## Getting started

To get started simply clone the kubespray repo then you'll need to make a couple edits and you'll be on your way!

```
git clone https://github.com/kubernetes-incubator/kubespray
cd kubespray
cp -R inventory/sample inventory/mycluster
```

The most important line is line #2 in the `inventory/mycluster/group_vars/all.yml`. You'll need to change the "none" value to the appropriate operating system that you're deploying to:

```
# Valid bootstrap options (required): ubuntu, coreos, centos, none
bootstrap_os: centos
```

##### Setting up your inventory file (hosts.ini):

```ini

k8-master-01 ansible_host=5.0.2.104 ansible_port=22 ansible_connection=ssh ansible_user=centos
k8-node-01 ansible_host=5.0.2.65 ansible_port=22 ansible_connection=ssh ansible_user=centos
k8-node-02 ansible_host=5.0.2.66 ansible_port=22 ansible_connection=ssh ansible_user=centos

[kube-master]

k8-master-01

[kube-nodes]

k8-node-01
k8-node-02

[k8s-cluster:children]

kube-nodes
kube-master

```

##### _Testing Connectivity_
```
$ ansible all -i hosts.ini -m ping --become

k8-node-01 | SUCCESS => {
"changed": false,
"ping": "pong"
}
k8-master-01 | SUCCESS => {
"changed": false,
"ping": "pong"
}
k8-node-02 | SUCCESS => {
"changed": false,
"ping": "pong"
}
```
Make sure you have connectivity to your machines sorted out and you're ready to roll.

## Installation

If all goes well, it shouldn't take but one command to fire things off. The entire provisioning process takes between 5-10 minutes generally so be patient and keep an eye on the ansible outputs.

```
$ ansible-playbook -i inventory/sample/hosts.ini cluster.yml -b -v
```

## Common Problems

I've come across, and helped support, deploying to a multitude of environments finding that almost all documentation is missing these few nuggets that result in doom & gloom.

#### Error from missing `python-netaddr` package

The documentation doesn't tell you that you'll need to have the `python-netaddr` package installed before provisioning. You'll end up with the following error:

```
TASK [kubespray-defaults : Configure defaults] ******************************************************************************************************************************************************************************************************************************************************************************
Monday 19 March 2018  02:29:37 +0000 (0:00:00.502)       0:00:08.900 **********
fatal: [localhost]: FAILED! => {"msg": "{u'no_proxy': u'{{ no_proxy }}', u'https_proxy': u\"{{ https_proxy| default ('') }}\", u'http_proxy': u\"{{ http_proxy| default ('') }}\"}: {%- if loadbalancer_apiserver is defined -%} {{ apiserver_loadbalancer_domain_name| default('') }}, {{ loadbalancer_apiserver.address | d
efault('') }}, {%- endif -%} {%- for item in (groups['k8s-cluster'] + groups['etcd'] + groups['calico-rr']|default([]))|unique -%} {{ hostvars[item]['access_ip'] | default(hostvars[item]['ip'] | default(hostvars[item]['ansible_default_ipv4']['address'])) }}, {%-   if (item != hostvars[item]['ansible_hostname']) -%}
{{ hostvars[item]['ansible_hostname'] }}, {{ hostvars[item]['ansible_hostname'] }}.{{ dns_domain }}, {%-   endif -%} {{ item }},{{ item }}.{{ dns_domain }}, {%- endfor -%} 127.0.0.1,localhost: The ipaddr filter requires python-netaddr be installed on the ansible controller"}
```

Installing:

```
sudo apt-get install python-netaddr
yum install python-netaddr
```

#### Error resulting from swap being enabled

```
TASK [kubernetes/preinstall : Stop if swap enabled] *************************************************************************************************************************************************************************************************************************************************************************
Monday 19 March 2018  02:32:51 +0000 (0:00:00.034)       0:00:11.243 **********

fatal: [localhost]: FAILED! => {
    "assertion": "ansible_swaptotal_mb == 0",
    "changed": false,
    "evaluated_to": false
}
```
Disabling swap:
```
[yomateod@centos-1 kubespray]$ sudo swapoff -a
[yomateod@centos-1 kubespray]$ free -m
              total        used        free      shared  buff/cache   available
Mem:           7318        2541        3342          20        1434        4466
Swap:             0           0           0

```


#### Installing kubectl on master

Install kubectl on the master node:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```
