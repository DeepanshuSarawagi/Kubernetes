# Follow below steps to install Kubernetes Multi-Node cluster using kubeadm

## Prerequisites

1. Create a free developer Redhat account
2. Download the RHEL7 iso DVD image (Since RHEL7 is system requirement for setting up calico CNI)
3. Create three VMs as follows with at least 2GB RAM and 2vCPUs
    - Kubernetes-Controller
    - Kubernetes-Node1
4. Once created, create a user and add it to %wheel group for sudo access
5. Update /etc/hostname (to a meaningful name) and /etc/hosts entries to enable communication between above three hosts
   - kubernetescontroller.dev.com
   - kubernetesnode1.dev.com
6. You need to subscribe in-order for your OS to receive updates.
   - $subscription-manager register --auto-attach
7. Disable swap on RHEL
   - $sudo swapoff -a
   - Edit /etc/fstab entry to disable swap - $sudo vi /etc/fstab
   - $reboot
8. Disable firewalld service on all the nodes
   - $ sudo systemctl stop firewalld
   - $ sudo systemctl disable firewalld

## We will install [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) tool using following instructions as outlined
## 1. Install Docker container

Installing docker container or any container runtime of your choice is mandatory.
For simplicity, we will install Docker on RHEL using [following instructions](https://docs.docker.com/engine/install/rhel/).

### Note:
#### OS requirements
To install Docker Engine, you need a maintained version of RHEL 7, RHEL 8 or RHEL 9 on s390x (IBM Z). Archived versions aren’t supported or tested.

a. Remove older version of Docker Engine
   - sudo yum remove docker \\\
     docker-client \\\
     docker-client-latest \\\
     docker-common \\\
     docker-latest \\\
     docker-latest-logrotate \\\
     docker-logrotate \\\
     docker-engine \\\
     podman \\\
     runc

b. Set up the repository
   - $sudo yum install -y yum-utils
   - $sudo yum-config-manager \\\
     --add-repo \\\
     https://download.docker.com/linux/centos/docker-ce.repo (We are updating the manager to centos package manager due to [architecture issue](https://access.redhat.com/discussions/6249651).)
   - $sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
   - $sudo systemctl start docker
   - $sudo systemctl status docker

### Note:

If you get below error while installing docker, you can follow this [fix](https://stackoverflow.com/questions/65878769/cannot-install-docker-in-a-rhel-server).

Error: Package: docker-ce-rootless-extras-20.10.18-3.el7.x86_64 (docker-ce-stable)
Requires: fuse-overlayfs >= 0.7
Error: Package: containerd.io-1.6.8-3.1.el7.x86_64 (docker-ce-stable)
Requires: container-selinux >= 2:2.74
Error: Package: docker-ce-rootless-extras-20.10.18-3.el7.x86_64 (docker-ce-stable)
Requires: slirp4netns >= 0.4
Error: Package: 3:docker-ce-20.10.18-3.el7.x86_64 (docker-ce-stable)
Requires: container-selinux >= 2:2.74
You could try using --skip-broken to work around the problem
You could try running: rpm -Va --nofiles --nodigest

## 2. Install [cri-dockerd](https://github.com/Mirantis/cri-dockerd), following the instructions in that source code repository.
This adapter provides a shim for Docker Engine that lets you control Docker via the Kubernetes Container Runtime Interface.

## 3. Install Kubectl
Next, we need to install kubectl following this [document](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management).

Remember, we do not have our control-pane initialized hence following command will not return anything.

$kubectl cluster-info

## 4. Install kubeadm, kubelet

Now, let us install kudeadm, kubelet. Follow this [document](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) carefully.

### 4a. Use kubeadm to create a cluster

To create a cluster using kubeadm, follow this [document](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/).

Remember to run the following command on controller node. Since, that is where your controller will run.

- $kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=192.168.77.132 --cri-socket unix:///var/run/cri-dockerd.sock

You will see an output similar to this.

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.77.132:6443 --token --discovery-token-ca-cert-hash sha256:abcdefghijkl

### 4b. Run below commands as non-root user

- $mkdir -p $HOME/.kube
- sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
- sudo chown $(id -u):$(id -g) $HOME/.kube/config

## 5. Install calico CNI

We need to perform this step to create the network which will enable the communication between controller and worker nodes.

- $https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml
- $kubectl create -f tigera-operator.yaml
- $https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/custom-resources.yaml
- Edit the cidr range in custom-resources.yaml 
  - We need to specify the same cidr range as what we specified in the kubeadm command. I chose 10.244.0.0/16
  - Snippet from my file
  ![calico custom resource yaml](https://github.com/DeepanshuSarawagi/Kubernetes/blob/main/Kubernetes-Cluster/img.png)

- $kubectl create -f custom-resources.yaml

Once the above commands are run, you will see controller-node in Ready state.

[dsarawagi@kubernetescontroller ~]$ kubectl get nodes

NAME                                     STATUS   ROLES           AGE   VERSION

kubernetescontroller.dev.deepanshu.com   Ready    control-plane   30m   v1.25.2

## 5a. Now it is time to join the worker nodes our network.

Execute below command on each worker node as root user.

- $kubeadm join 192.168.77.132:6443 --token "token" --discovery-token-ca-cert-hash sha256:abcdefghijkl --cri-socket unix:///var/run/cri-dockerd.sock

## 6. Create PODs to run on worker nodes

Use this yaml definition to create [nginx](../Pods/nginx.yaml) POD on worker node using below command.
- $ kubectl create -f nginx.yaml

You will notice that PODs are running on worker node. To validate the same, you can execute below command which will specify

on which node, pod is running.

- $kubectl get pods -o wide
