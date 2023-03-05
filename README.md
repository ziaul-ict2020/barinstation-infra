# brs2

# Instal LXD
 
sudo snap install lxd
 
 sudo lxd init
 
 Would you like to use LXD clustering? (yes/no) [default=no]: no
 
Do you want to configure a new storage pool? (yes/no) [default=yes]: yes

Name of the new storage pool [default=default]:

Name of the storage backend to use (dir, lvm, zfs, ceph, btrfs) [default=zfs]:

Create a new ZFS pool? (yes/no) [default=yes]: yes

Would you like to use an existing empty block device (e.g. a disk or partition)?(yes/no) [default=no]: no

Size in GB of the new loop device (1GB minimum) [default=9GB]: 30GB

Would you like to connect to a MAAS server? (yes/no) [default=no]: no

Would you like to create a new local network bridge? (yes/no) [default=yes]: yes

What should the new bridge be called? [default=lxdbr0]:

What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: auto

What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none

What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none

What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none

Would you like the LXD server to be available over the network? (yes/no) [default=no]:

Would you like stale cached images to be updated automatically? (yes/no) [default=yes]: yes

Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: no


# profile create

lxc profile create kubernetes
lxc profile edit kubernetes

#config:
  boot.autostart: "true"
  linux.kernel_modules: ip_vs,ip_vs_rr,ip_vs_wrr,ip_vs_sh,ip_tables,ip6_tables,netlink_diag,nf_nat,overlay,br_netfilter
  raw.lxc: |
    lxc.apparmor.profile=unconfined
    lxc.mount.auto=proc:rw sys:rw cgroup:rw
    lxc.cgroup.devices.allow=a
    lxc.cap.drop=
  security.nesting: "true"
  security.privileged: "true"
description: ""
devices:
  aadisable:
    path: /sys/module/nf_conntrack/parameters/hashsize
    source: /sys/module/nf_conntrack/parameters/hashsize
    type: disk
  aadisable1:
    path: /sys/module/apparmor/parameters/enabled
    source: /dev/null
    type: disk
  aadisable3:
    path: /dev/kmsg
    source: /dev/kmsg
    type: disk
name: kubernetes
used_by: []

# Create Container

lxc launch --profile default --profile kubernetes ubuntu:20.04 master

lxc launch --profile default --profile kubernetes ubuntu:20.04 worker1

lxc launch --profile default --profile kubernetes ubuntu:20.04 worker2

lxc list


![image](https://user-images.githubusercontent.com/71640997/192134252-08a6d79c-0340-4d33-80b8-de5200d36f72.png)


#set password

lxc exec master bash

lxc exec worker1 bash

lxc exec worker2 bash

# Kubernetes cluster using Ansible

ansible-playbook user.yml

ansible-playbook kubernetes-inatll.yaml

ansible-playbook masternode.yaml

ansible-playbook workerjoin.yaml

# Kubernetes Dashboard and metric server

kubectl create -f dashboard.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml

kubectl patch -n kubernetes-dashboard svc kubernetes-dashboard --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'

#get the token to login the dashboard

kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"


kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


 
