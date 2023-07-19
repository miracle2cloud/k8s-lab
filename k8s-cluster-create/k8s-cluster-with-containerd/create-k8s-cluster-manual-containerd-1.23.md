### Create 1 master and 2 worker node vms for the cluster(Ubuntu)
### For GCP as below
### GCP-->gcloud commands to create 1 master and 2 worker node vms for the cluster(Ubuntu)
```bash

gcloud compute instances create master worker-1 worker-2 --create-disk=auto-delete=yes,boot=yes,image=projects/ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20230302 --zone us-central1-a --machine-type=e2-medium
```

### Create your own user in Ubuntu follow the below steps:
```bash
$ adduser <username>

#Add the new user to the sudo group 
$ usermod -aG sudo <username>

Switch to newly created user:
$ su - <username>

#How to Enable SSH Password Authentication
#To enable SSH password authentication, you must SSH in as root to edit this file:
$ sudo vi /etc/ssh/sshd_config

PasswordAuthentication yes
sudo service ssh restart
```
## Install containerd on all nodes (`Both master and worker`)
### Disable swap
  ```bash
  $ sudo swapoff -a
  $ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
  ```
### Load the following kernel modules on all the nodes
  ```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

  $ sudo modprobe overlay
  $ sudo modprobe br_netfilter
  ```

### Set the following Kernel parameters for Kubernetes to configure sysctl to persist across system reboots
  ```bash
  $ sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
    EOF
  ```

 ### Reload the above changes
   ```bash
   $ sudo sysctl --system
   ```
 ### Install containerd packages, run time, restart and enable.
   ```bash
    $ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    $ sudo apt update
    $ sudo apt install -y containerd.io
    $ sudo mkdir -p /etc/containerd
    $ sudo bash -c "containerd config default > /etc/containerd/config.toml"
    $ sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
    $ sudo systemctl restart containerd
    $ sudo systemctl enable containerd 
   ```
## Install K8S components Kubectl, kubeadm & kubelet(all nodes)
 ### Add apt repository for Kubernetes
 ```bash
 sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
 echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
 ```
 ### Install K8s components
 ```bash
 
 $ sudo apt update
 $ sudo apt install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00
 $ sudo systemctl enable kubelet
 $ sudo apt-mark hold kubelet kubeadm kubectl
 
 ```
### Initialize Kubernetes cluster with Kubeadm command (only Master)
 ```bash
 $ sudo kubeadm config images pull --cri-socket /run/containerd/containerd.sock --kubernetes-version v1.23.0
 $ sudo kubeadm init --upload-certs --kubernetes-version=v1.23.0  --control-plane-endpoint=$(hostname) --ignore-preflight-errors=all  --cri-socket /run/containerd/containerd.sock
 $ sudo mkdir -p $HOME/.kube
 $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 $ sudo chown $(id -u):$(id -g) $HOME/.kube/config
 ```
### Install Weave Net network plugin (only Master)
```bash
#Install CNI so that pods can communicate across nodes and also Cluster DNS to start functioning. Apply weave CNI (Container Network Interface) on the master node
$ kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
$ kubectl get nodes

#If you need to generate a new tokem use the below command (Optional Not required , if you have the above token generated)

  $ sudo kubeadm token create --print-join-command
  
### Execute Join command to the cluster from the above token received(only on Worker Nodes)
  $ sudo kubeadm join master:6443 --token <received-token-above>
