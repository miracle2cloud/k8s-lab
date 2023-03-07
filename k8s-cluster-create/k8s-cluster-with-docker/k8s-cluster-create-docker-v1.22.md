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
usermod -aG sudo <username>

Switch to newly created user:
su - <username>

#How to Enable SSH Password Authentication
#To enable SSH password authentication, you must SSH in as root to edit this file:
sudo vi /etc/ssh/sshd_config

PasswordAuthentication yes

sudo service ssh restart

```

### Install Docker on all nodes (`Both master and worker`)

[Installing Docker on ubuntu](https://docs.docker.com/engine/install/ubuntu/)
```bash
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce=5:19.03.12~3-0~ubuntu-bionic -y
sudo apt-mark hold docker-ce
sudo usermod -aG docker <username>
sudo docker version

```

### On all nodes, install kubeadm, kubelet, and kubectl. (`Both master and worker`)
[Refer Here for Official documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
```bash
# Update the apt package index and install packages needed to use the Kubernetes apt repository:
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Download the Google Cloud public signing key:
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# Add the Kubernetes apt repository:
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update apt package index, install kubelet, kubeadm and kubectl with a specific version, and pin their version:
sudo apt-get update
sudo apt-get install -y kubelet=1.22.1-00 kubeadm=1.22.1-00 kubectl=1.22.1-00
sudo apt-mark hold kubelet kubeadm kubectl
```

### On the Kube master server (` Master Only `)
```bash
# Configure cgroup driver used by kubelet on control-plane.(Only on master node)
	sudo docker info | grep -i cgroup

Create a below config file on master and save.
vi kubeadm-config.yaml
----------------------
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.22.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
#cgroupDriver: systemd
cgroupDriver: cgroupfs
    
  # Initialising the control-plane node run the below command on the (master node)
  sudo kubeadm init --config kubeadm-config.yaml

  sudo sed -i "s/cgroupDriver: systemd/cgroupDriver: cgroupfs/g" /var/lib/kubelet/config.yaml
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet

#Set up local kubeconfig:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# If you need to generate a new tokem use the below command (Optional Not required , if you have the above token generated)
    sudo kubeadm token create --print-join-command
```

### Join the Worker nodes (`Only on Worker Nodes`)
```bash
Execute the command that is been generated as the output of the previous command, where we got the token generated. 
It will be similar to the below command

sudo kubeadm join 10.18.0.30:6443 --token 06fdx2qa.v2jxfdfdflapu54gi3s41 --discovery-token-ca-cert-hash sha256:ed92e6bdfd6d7e27abc8f9247d6de33a7dfd56b57a250195d57647bf3138c9a4e7d7a8
```


#### Come back to Master node and executhe below commands (` Only on Master Node `)
```bash

# Verify the cluster status by executing kubectl command on the master node
kubectl get nodes

#Install CNI so that pods can communicate across nodes and also Cluster DNS to start functioning. Apply weave CNI (Container Network Interface) on the master node
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
kubectl get nodes

```
