# Install kafka kubernetes Cluster
Author : Mansur Ul Hasan 
Email : mansurali901@gmail.com

Environment :
- Three nodes cluster with 1 Master and 2 slave with tainted
Kubernetes Cluster Setup Steps....
Step # 1 Install Docker 

apt-get update && apt-get install -qy docker.io

Step # 2 Add repository 

sudo apt-get update; apt-get install -y apt-transport-https; curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list && sudo apt-get update 

Step # 3 Install Kubernetes components 
sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubernetes-cni

Step # 4 Initialize Kubernetes Cluster 
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=${YOUR MASTER IP}

Step # 5 Configure Environment 

cd $HOME
sudo whoami
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
echo "export KUBECONFIG=$HOME/admin.conf" | tee -a ~/.bashrc

Step # 6 Add Networking
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

Step # 7 Tainting Nodes
kubectl taint nodes --all dedicated-

Step # 8 Check Status 
kubectl get all --namespace=kube-system

# Setup Kafka in kubernetes Cluster
