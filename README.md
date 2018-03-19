# My Awesome Book

This file file serves as your book's preface, a great place to describe your book's content and ideas.

```
sudo apt-get install python-netaddr
yum install python-netaddr
```

#### Inventory file (hosts.ini):
```ini

k8-master-01    ansible_host=5.0.2.104 ansible_port=22 ansible_connection=ssh  ansible_user=centos
k8-node-01      ansible_host=5.0.2.65 ansible_port=22 ansible_connection=ssh  ansible_user=centos
k8-node-02      ansible_host=5.0.2.66 ansible_port=22 ansible_connection=ssh  ansible_user=centos

[kube-master]

    k8-master-01

[kube-nodes]

    k8-node-01
    k8-node-02

[k8s-cluster:children]

    kube-nodes
    kube-master

```

#### Test Connectivity
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