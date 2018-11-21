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
wget https://github.com/rancher/rke/releases/download/v0.1.11/rke_linux-amd64; chmod +x rke_linux-amd64; 
sudo cp rke_linux-amd64 /usr/local/bin

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

$ kubectl get pods -o wide --sort-by="{.spec.nodeName}" --all-namespaces

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

```
Troubleshooting:
$ kubectl logs default-http-backend-797c5bc547-grdfd -n ingress-nginx
Error from server (BadRequest): container "default-http-backend" in pod "default-http-backend-797c5bc547-grdfd" is not available
$ kubectl -n ingress-nginx  describe rs/default-http-backend-797c5bc547
```

```
Example setup single node (provision.sh ---> mem = 2048, cpu = 2)


$ ./provision
$ virsh console k8s-prod-1 (complete ks installation ---> disk setup)
$ virsh start k8s-prod-1
$ ssh rke@k8s-prod-1 
 - setup hosts file add 192.168.122.111 k8s-prod-1
$ rke up

$ export KUBECONFIG=./kube_config_cluster.yml

$ kubectl get pods -o wide --sort-by="{.spec.nodeName}" --all-namespaces
NAMESPACE       NAME                                      READY   STATUS      RESTARTS   AGE   IP                NODE              NOMINATED NODE
ingress-nginx   default-http-backend-797c5bc547-5bmb6     1/1     Running     0          5m    10.42.0.5         192.168.122.111   <none>
ingress-nginx   nginx-ingress-controller-zcx7m            1/1     Running     0          5m    192.168.122.111   192.168.122.111   <none>
kube-system     canal-m8mz9                               3/3     Running     0          6m    192.168.122.111   192.168.122.111   <none>
kube-system     kube-dns-7588d5b5f5-p76dx                 3/3     Running     0          6m    10.42.0.3         192.168.122.111   <none>
kube-system     kube-dns-autoscaler-5db9bbb766-trjls      1/1     Running     0          6m    10.42.0.2         192.168.122.111   <none>
kube-system     metrics-server-97bc649d5-t242w            1/1     Running     0          5m    10.42.0.4         192.168.122.111   <none>
kube-system     rke-ingress-controller-deploy-job-l69nx   0/1     Completed   0          5m    192.168.122.111   192.168.122.111   <none>
kube-system     rke-kubedns-addon-deploy-job-55qnm        0/1     Completed   0          6m    192.168.122.111   192.168.122.111   <none>
kube-system     rke-metrics-addon-deploy-job-4l2k6        0/1     Completed   0          5m    192.168.122.111   192.168.122.111   <none>
kube-system     rke-network-plugin-deploy-job-np7xz       0/1     Completed   0          6m    192.168.122.111   192.168.122.111   <none>

$ ./install_rancher_server.sh

$ kubectl get pods -o wide --sort-by="{.spec.nodeName}" --all-namespaces
NAMESPACE       NAME                                      READY   STATUS      RESTARTS   AGE   IP                NODE              NOMINATED NODE
cattle-system   rancher-79977f9d95-7pxct                  1/1     Running     1          4m    10.42.0.10        192.168.122.111   <none>
cattle-system   rancher-79977f9d95-p5jwd                  1/1     Running     1          4m    10.42.0.8         192.168.122.111   <none>
cattle-system   rancher-79977f9d95-z9jl7                  1/1     Running     2          4m    10.42.0.9         192.168.122.111   <none>
ingress-nginx   default-http-backend-797c5bc547-5bmb6     1/1     Running     0          17m   10.42.0.5         192.168.122.111   <none>
ingress-nginx   nginx-ingress-controller-zcx7m            1/1     Running     0          17m   192.168.122.111   192.168.122.111   <none>
kube-system     canal-m8mz9                               3/3     Running     0          17m   192.168.122.111   192.168.122.111   <none>
kube-system     cert-manager-787d565b77-k2gk8             1/1     Running     0          4m    10.42.0.7         192.168.122.111   <none>
kube-system     kube-dns-7588d5b5f5-p76dx                 3/3     Running     0          17m   10.42.0.3         192.168.122.111   <none>
kube-system     kube-dns-autoscaler-5db9bbb766-trjls      1/1     Running     0          17m   10.42.0.2         192.168.122.111   <none>
kube-system     metrics-server-97bc649d5-t242w            1/1     Running     0          17m   10.42.0.4         192.168.122.111   <none>
kube-system     rke-ingress-controller-deploy-job-l69nx   0/1     Completed   0          17m   192.168.122.111   192.168.122.111   <none>
kube-system     rke-kubedns-addon-deploy-job-55qnm        0/1     Completed   0          17m   192.168.122.111   192.168.122.111   <none>
kube-system     rke-metrics-addon-deploy-job-4l2k6        0/1     Completed   0          17m   192.168.122.111   192.168.122.111   <none>
kube-system     rke-network-plugin-deploy-job-np7xz       0/1     Completed   0          17m   192.168.122.111   192.168.122.111   <none>
kube-system     tiller-deploy-57f988f854-w4f4j            1/1     Running     0          5m    10.42.0.6         192.168.122.111   <none>


```

Access RKE console 

https://k8s-prod-1 -> cluster local 




