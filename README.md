```
                                                         __                 __
                            __  ______  ____ ___  ____ _/ /____  ____  ____/ /
                           / / / / __ \/ __ `__ \/ __ `/ __/ _ \/ __ \/ __  /
                          / /_/ / /_/ / / / / / / /_/ / /_/  __/ /_/ / /_/ /
                          \__, /\____/_/ /_/ /_/\__,_/\__/\___/\____/\__,_/
                         /____                     matthewdavis.io, holla!

```

[![Clickity click](https://img.shields.io/badge/k8s%20by%20example%20yo-limit%20time-ff69b4.svg?style=flat-square)](https://k8.matthewdavis.io) [![Twitter Follow](https://img.shields.io/twitter/follow/yomateod.svg?label=Follow&style=flat-square)](https://twitter.com/yomateod) [![Skype Contact](https://img.shields.io/badge/skype%20id-appsoa-blue.svg?style=flat-square)](skype:appsoa?chat)

# Rollin deep with Kubespray

To get started simply clone the kubespray repo then you'll need to make a couple edits and you'll be on your way!

## Getting started

You'll want to use a linux based host to perform your deployment from and will need ansible installed locally (`pip install ansible`).

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

##### _Installing kubectl on master_

Install kubectl on the master node:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```
Now query to make sure you can get to your resources:`
```
[centos@k8-master-01 ~]$ ./kubectl get node,pod,svc --all-namespaces -o wide
NAME              STATUS    ROLES     AGE       VERSION           EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
no/k8-master-01   Ready     master    1h        v1.9.3+coreos.0   <none>        CentOS Linux 7 (Core)   3.10.0-693.21.1.el7.x86_64   docker://17.12.1-ce
no/k8-node-01     Ready     node      1h        v1.9.3+coreos.0   <none>        CentOS Linux 7 (Core)   3.10.0-693.21.1.el7.x86_64   docker://17.12.1-ce
no/k8-node-02     Ready     node      1h        v1.9.3+coreos.0   <none>        CentOS Linux 7 (Core)   3.10.0-693.21.1.el7.x86_64   docker://17.12.1-ce

NAMESPACE     NAME                                       READY     STATUS    RESTARTS   AGE       IP               NODE
kube-system   po/calico-node-5zbkq                       1/1       Running   0          1h        5.0.2.66         k8-node-02
kube-system   po/calico-node-mlwrr                       1/1       Running   0          1h        5.0.2.65         k8-node-01
kube-system   po/calico-node-rm2d4                       1/1       Running   1          1h        5.0.2.104        k8-master-01
kube-system   po/kube-apiserver-k8-master-01             1/1       Running   1          1h        5.0.2.104        k8-master-01
kube-system   po/kube-controller-manager-k8-master-01    1/1       Running   1          1h        5.0.2.104        k8-master-01
kube-system   po/kube-dns-79d99cdcd5-bflc7               3/3       Running   0          1h        10.233.117.130   k8-node-01
kube-system   po/kube-dns-79d99cdcd5-td4n8               3/3       Running   0          1h        10.233.65.65     k8-node-02
kube-system   po/kube-proxy-k8-master-01                 1/1       Running   1          1h        5.0.2.104        k8-master-01
kube-system   po/kube-proxy-k8-node-01                   1/1       Running   0          1h        5.0.2.65         k8-node-01
kube-system   po/kube-proxy-k8-node-02                   1/1       Running   0          1h        5.0.2.66         k8-node-02
kube-system   po/kube-scheduler-k8-master-01             1/1       Running   1          1h        5.0.2.104        k8-master-01
kube-system   po/kubedns-autoscaler-5564b5585f-fnv8n     1/1       Running   0          1h        10.233.117.129   k8-node-01
kube-system   po/kubernetes-dashboard-69cb58d748-l97m6   1/1       Running   1          1h        10.233.116.194   k8-master-01
kube-system   po/nginx-proxy-k8-node-01                  1/1       Running   0          1h        5.0.2.65         k8-node-01
kube-system   po/nginx-proxy-k8-node-02                  1/1       Running   0          1h        5.0.2.66         k8-node-02

NAMESPACE     NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE       SELECTOR
default       svc/kubernetes             ClusterIP   10.233.0.1     <none>        443/TCP         1h        <none>
kube-system   svc/kube-dns               ClusterIP   10.233.0.3     <none>        53/UDP,53/TCP   1h        k8s-app=kube-dns
kube-system   svc/kubernetes-dashboard   ClusterIP   10.233.31.36   <none>        443/TCP         1h        k8s-app=kubernetes-dashboard
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
You'll want to edit your `/etc/fstab` file on each node affected to make sure that the changes survive reboots.
