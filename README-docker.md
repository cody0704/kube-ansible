# Kube-Ansible

Fork: ![kairen/kube-ansible](https://github.com/kairen/kube-ansible)

## Hardware/Software

1. cpu/ram
	* master: 2c4g ↑
	* node: 1c2g ↑
2. OS:
  * linux (CentOS7/RHEL7)
  	* kernel 3.10 ↑

## Installation

### 事前準備

* 所有節點的網路之間可以互相溝通
* 部署節點對其他節點不需要 SSH 密碼即可登入
* 所有節點都擁有 Sudoer 權限，並且不需要輸入密碼
* 所有節點需要安裝 Python
* 所有節點需要設定 /etc/host 解析到所有主機
* 部署節點需要安裝 Ansible

## Hostname

```
hostnamectl set-homename kube01
hostnamectl set-homename kube02
```

## Hosts

```bash
vim /etc/hosts
```

```
192.168.0.122   kube01
192.168.0.116   kube02
```

### NTP

```bash
sudo yum install ntp -y
```

```bash
sudo systemctl start ntpd
sudo systemctl enable ntpd
```

## SSH PubeKey

```bash
ssh-keygen -t rsa -P ""

ssh-copy-id 192.168.0.122
ssh-copy-id 192.168.0.116
```

## Firewall

> Disable the firewall or open the service port on the master node

```
systemctl disable firewalld && systemctl stop firewalld
```

## Install Ansible

### Install Package

```bash
sudo yum install -y epel-release
sudo yum install -y ansible python-netaddr cowsay git
```

### ansible version

```bash
ansible --version
```

## Install Kube Cluster

### Git Download

```bash
git clone https://github.com/codychen0704/kube-ansible.git
cd kube-ansible
```

### Setting

#### inventory/hosts.ini

```
[etcds]
kube01 ansible_user=root

[masters]
kube01 ansible_user=root

[nodes]
kube02 ansible_user=root

[kube-cluster:children]
masters
nodes
```

> ssh password / sudo password

```
[kube-cluster:vars]
ansible_ssh_pass=12345678 ansible_become_pass=12345678
```

#### `group_vars/all.yml`

```
kube_version: 1.14.3

container_runtime: docker

cni_enable: true
container_network: calico
cni_iface: ""

vip_interface: ""
vip_address: 192.168.0.122

etcd_iface: ""

enable_ingress: true
enable_dashboard: true
enable_logging: true
enable_monitoring: true
enable_metric_server: true

grafana_user: "admin"
grafana_password: "p@ssw0rd"
```

### Check

透過 Ansible 檢查叢集狀態

```
ansible -i inventory/hosts.ini all -m ping
```

狀態無異常，開始安裝 k8s cluster

```
ansible-playbook -i inventory/hosts.ini cluster.yml
```

透過其中一台 master 確認 cluster 狀態

```
# k8s components status
$ kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}

# k8s node status
$ kubectl get no
NAME      STATUS    ROLES     AGE       VERSION
kube01    Ready     master    5m        v1.14.3
kube02    Ready     <none>    3m        v1.14.3
```

## Addone

部署 Kubernetes extra addons

```
ansible-playbook -i inventory/hosts.ini addons.yml
```

完成後即可透過 kubectl 來檢查服務，如 kubernetes-dashboard

```
kubectl get po,svc -n kube-system -l k8s-app=kubernetes-dashboard
```

## HA

```
export PKI="/etc/kubernetes/pki/etcd"
ETCDCTL_API=3 etcdctl \
    --cacert=${PKI}/etcd-ca.pem \
    --cert=${PKI}/etcd.pem \
    --key=${PKI}/etcd-key.pem \
    --endpoints="https://192.168.0.122:2379" \
    member list

_____________, started, kube01, https://192.168.0.122:2380, https://192.168.0.122:2379
_____________, started, kube01, https://192.168.0.127:2380, https://192.168.0.122:2379
_____________, started, kube01, https://192.168.0.122:2380, https://192.168.0.122:2379
