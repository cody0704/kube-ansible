# Kube-Ansible

Fork: [kairen/kube-ansible](https://github.com/kairen/kube-ansible)

目錄

* Containerd: [Containerd](https://github.com/cody0704/kube-ansible/blob/master/README-containerd.md)
* Docker: [Docker](https://github.com/cody0704/kube-ansible/blob/master/README-docker.md)

# 提升效能

## swap

```
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```

## selinux

```
# 允許 containers 連到 host
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```