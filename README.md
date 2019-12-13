# Kube-Ansible

Fork: ![kairen/kube-ansible](https://github.com/kairen/kube-ansible)

目錄

* Containerd: ![Containerd](#)
* Docker: ![Docker](#)

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