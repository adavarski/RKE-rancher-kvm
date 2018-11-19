# Rancher setup: kvm
A easy way to get a Rancher Kubernetes cluster up and running on KVM/Libvirt

This script will create machines in KVM prepared with docker and ssh key. It will also generate a cluster.yml that can be used by RKE to provision a Kubernetes cluster. This cluster can then be joined to a Rancher manager UI.

Create 3 nodes by running 
```
Download CentOS ISO ---> ./cdimages/CentOS-7-x86_64-Minimal-1804.iso
./provision.sh 3
```

You will end up with 3 virtual machines having a user named rke with the SSH keys found in ~/.ssh/ on your host. Also, a cluster.yml will be generated.

By default, all nodes will be running etcd, controlplane and worker containers. Edit cluster.yml to change this to your liking. 

Download rke https://github.com/rancher/rke/releases/tag/v0.1.11

Then, simply run rke to create the cluster

```
rke up
```

When done, a cluster will be running. It will generate a config file you can use with kubectl.

```
kubectl get cs --kubeconfig kube_config_cluster.yml

export KUBECONFIG=./kube_config_cluster.yml

$ kubectl get node 
NAME              STATUS   ROLES                      AGE   VERSION
192.168.122.111   Ready    controlplane,etcd,worker   16m   v1.11.3
192.168.122.112   Ready    controlplane,etcd,worker   16m   v1.11.3

$ kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
canal-5rsvl                            3/3     Running   0          16m
canal-rfgh6                            3/3     Running   0          16m
kube-dns-7588d5b5f5-srqtl              3/3     Running   0          16m
kube-dns-autoscaler-5db9bbb766-sbwvr   1/1     Running   0          16m
metrics-server-97bc649d5-g7c7m         1/1     Running   0          16m
```

# Install Rancher UI
Install the Rancher Server (UI) via Helm with script: 
Fore using it, you need to download the Helm client and have it in your path.
```
vi install_rancher_server.sh
# Change the last line with the hostname
# Add the hostname to your /etc/hosts on a worker node

./install_rancher_server.sh
```
It will create a ServiceAccound for Tiller and a ClusterRoleBinding for this ServiceAccount to a ClusteRole called cluster-admin. Then it installs Tiller, adds the repository for Rancher Server and installs Rancher and cert-manager.
