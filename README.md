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

# metric server

wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability.yaml

vim high-availability.yaml
add this - --kubelet-insecure-tls into the args section 

      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: k8s.gcr.io/metrics-server/metrics-server:v0.6.1

kubectl apply -f high-availability.yaml

Also, to maximize the efficiency of this highly available configuration, it is recommended to add the --enable-aggregator-routing=true CLI flag to the kube-apiserver so that requests sent to Metrics Server are load balanced between the 2 instances.

root@master01:~# cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep aggregator
    - --enable-aggregator-routing=true

Kubernets Dashboard:
====================

I want to deploy kubernets dashboard on master node . For that we need to set label on all master node

kubectl label nodes master01 access=kubernets-dashboard
kubectl label nodes master02 access=kubernets-dashboard
kubectl label nodes master03 access=kubernets-dashboard

kubectl taint nodes master01 node-role.kubernetes.io/master:NoSchedule
kubectl taint nodes master02 node-role.kubernetes.io/master:NoSchedule
kubectl taint nodes master03 node-role.kubernetes.io/master:NoSchedule

kubectl get nodes --show-labels | grep access=kubernets-dashboard

master01   Ready    control-plane   39d   v1.24.3   access=kubernets-dashboard,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,istio=ingressgateway,kubernetes.io/arch=amd64,kubernetes.io/hostname=master01,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=,type=calico-kube-controller
master02   Ready    control-plane   39d   v1.24.3   access=kubernets-dashboard,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,istio=ingressgateway,kubernetes.io/arch=amd64,kubernetes.io/hostname=master02,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=,type=calico-kube-controller
master03   Ready    control-plane   39d   v1.24.3   access=kubernets-dashboard,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,istio=ingressgateway,kubernetes.io/arch=amd64,kubernetes.io/hostname=master03,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=,type=calico-kube-controller


mkdir kubernets-dashboard
cd kubernets-dashboard
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml

vim recommended.yaml

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  replicas: 3

...
      nodeSelector:
        access: kubernets-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          effect: NoSchedule
        - key: node-role.kubernetes.io/master
          effect: NoSchedule

kubectl apply -f recommended.yaml

kubectl -n kubernetes-dashboard get all

NAME                                            READY   STATUS    RESTARTS   AGE

pod/dashboard-metrics-scraper-8c47d4b5d-mlj29   1/1     Running   0          14d
pod/kubernetes-dashboard-c685fc6f-2khzd         1/1     Running   0          14d
pod/kubernetes-dashboard-c685fc6f-jpftg         1/1     Running   0          14d
pod/kubernetes-dashboard-c685fc6f-m8twg         1/1     Running   0          14d

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE

service/dashboard-metrics-scraper   ClusterIP   10.10.185.157   <none>        8000/TCP   14d
service/kubernetes-dashboard        ClusterIP   10.10.9.54      <none>        443/TCP    14d

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE

deployment.apps/dashboard-metrics-scraper   1/1     1            1           14d
deployment.apps/kubernetes-dashboard        3/3     3            3           14d

NAME                                                  DESIRED   CURRENT   READY   AGE

replicaset.apps/dashboard-metrics-scraper-8c47d4b5d   1         1         1       14d
replicaset.apps/kubernetes-dashboard-c685fc6f         3         3         3       14d

kubectl -n kubernetes-dashboard get sa

NAME                   SECRETS   AGE
default                0         14d
kubernetes-dashboard   0         14d
root@master01:~/dashboard# kubectl -n kubernetes-dashboard get secrets
NAME                              TYPE     DATA   AGE
kubernetes-dashboard-certs        Opaque   0      14d
kubernetes-dashboard-csrf         Opaque   1      14d
kubernetes-dashboard-key-holder   Opaque   2      14d
root@master01:~/dashboard# kubectl -n kubernetes-dashboard get role
NAME                   CREATED AT
kubernetes-dashboard   2022-08-26T21:11:11Z
root@master01:~/dashboard# kubectl -n kubernetes-dashboard get clusterrole | grep dashboard
kubernetes-dashboard                                                   2022-08-26T21:11:11Z
root@master01:~/dashboard#

Accessing the Kubernetes Dashboard:
===================================

We can access the Kubernetes dashboard in the following ways:

  1. kubectl port-forward (only from kubectl machine)
  2. kubectl proxy (only from kubectl machine)
  3. Kubernetes Service (NodePort/ClusterIp/LoadBalancer)
  4. Ingress Controller (Layer 7)

step 1: Using proxy
===================

kubectl proxy

Step 2: Using port-forward
===========================

kubectl -n kubernetes-dashboard port-forward services/kubernetes-dashboard 20021:443

Step 3: Using Nodeport
=======================
kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard

Once the file is opened, change the type of service from ClusterIP to NodePort and save the file as shown below. By default, the service is only available internally to the cluster (ClusterIP) but changing to NodePort exposes the service to the outside.

# Updated the type to NodePort in the service.
 ports:
 port: 443 
 protocol: TCP
 targetPort: 8443
 selector:
 k8s-app: kubernetes-dashboard
 sessionAffinity: None
 type: NodePort 


 Step 4: Using Kubernets Ingress controller
 ==========================================

 Here we are using istio ingress gateway for accessing kubernets dashboard from outside of cluster

 First create a gateway for kubernets dashboard
 ================================================
cat <<EOF>> dashboard-gateway.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: dashboard-gateway
  namespace: kubernetes-dashboard
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 443
      name: http
      protocol: HTTPS
    tls:
      mode: PASSTHROUGH
    hosts:
    - dashboard.ziaulhaque.com
EOF


Now create a virtual service

cat <<EOF>> dashboard.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: dashboard
  namespace: kubernetes-dashboard
spec:
  hosts:
  - dashboard.ziaulhaque.com
  gateways:
  - dashboard-gateway
  tls:
  - match:
    - port: 443
      sniHosts:
      - dashboard.ziaulhaque.com
    route:
    - destination:
        host: kubernetes-dashboard
        port:
          number: 443
EOF

Now check kubernets-dashboard using curl command from master node


export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export TCP_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].nodePort}')

curl -v --resolve "dashboard.ziaulhaque.com:$SECURE_INGRESS_PORT:$INGRESS_HOST" "https://dashboard.ziaulhaque.com:$SECURE_INGRESS_PORT"

Create an admin user to access dashboard
==========================================

cat <<EOF>> admin-user.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF





# kubectl -n kubernetes-dashboard get sa admin-user

NAME         SECRETS   AGE
admin-user   0         86s

#kubectl -n kubernetes-dashboard get clusterrolebindings.rbac.authorization.k8s.io | grep admin-user

admin-user                                             ClusterRole/cluster-admin                                                          91s
root@master01:~/dashboard#

Get the admin token using the command below.

root@master01:~# kubectl -n kubernetes-dashboard create token admin-user

eyJhbGciOiJSUzI1NiIsImtpZCI6IkxfM3BSZ0xWSHhjVW5PazQtR2dORDVGZlFZMnJtSjlkMFlaeTJ5SGNwazQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjYyODk1OTYxLCJpYXQiOjE2NjI4OTIzNjEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiY2IxNDY2NDctOWU3Ny00OWExLWIyNDUtNzlmY2FlMzcwN2UyIn19LCJuYmYiOjE2NjI4OTIzNjEsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.AqtgWEqedwffhGYxho6r15kxTNtK0Th76phN3V6DGQoR5Q0WGt5aj5lWOdSuDTD6kQd4ju75OYYBMsN7FD2LOjcPv-sTzArQLGQ9q_NG9lgcql4uSrNx8wQQwlKU0YkEJJZOz7Z9CRsJPmoEvv00bOSooBo_btm_Wv5sOmKu4I30aJI9opvH3h4viH3DHKAgAxqD9jMyyWIn4-y92uW9lpyeYQybhevZxTvD7xlHK3R7iGuSS5DQkEhLJa4xClOGwZPlGBcQ07O0rOVQIQNcrmOHc5iJ4XjCFEsj4YUHMz3Tqk-mKO8UIY9KSB8C3tGR0HEdmiebXT9ArDdGE3dvuA



kubectl get secret -n kubernetes-dashboard $(kubectl get serviceaccount admin-user -n kubernetes-dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
kubectl get secret -n kubernets-dashboard $(kubectl get serviceaccount admin-user -n kubernetes-dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode




root@master01:~/dashboard# cat dashboard-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-secret
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: admin-user
type: kubernetes.io/service-account-token
root@master01:~/dashboard#

kubectl -n kubernetes-dashboard get secrets admin-secret -o jsonpath={.data.token} | base64 -d \n