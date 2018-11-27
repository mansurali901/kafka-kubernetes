# Install kafka kubernetes Cluster
Author : Mansur Ul Hasan 
Email : mansurali901@gmail.com

Environment :
- Three nodes cluster with 1 Master and 2 slave with tainted
Kubernetes Cluster Setup Steps....
Step # 1 Install Docker 

.. apt-get update && apt-get install -qy docker.io

Step # 2 Add repository 

.. sudo apt-get update; apt-get install -y apt-transport-https; curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

.. echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list && sudo apt-get update 

Step # 3 Install Kubernetes components 
.. sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubernetes-cni

Step # 4 Initialize Kubernetes Cluster 
.. kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=${YOUR MASTER IP}

Step # 5 Configure Environment 

.. cd $HOME
.. sudo whoami
.. sudo cp /etc/kubernetes/admin.conf $HOME/
.. sudo chown $(id -u):$(id -g) $HOME/admin.conf
.. export KUBECONFIG=$HOME/admin.conf
.. echo "export KUBECONFIG=$HOME/admin.conf" | tee -a ~/.bashrc

Step # 6 Add Networking
.. kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

Step # 7 Tainting Nodes
.. kubectl taint nodes --all dedicated-

Step # 8 Check Status 
.. kubectl get all --namespace=kube-system

# Setup Kafka in kubernetes Cluster
We will have three replicas in each POD in our deployment 

- Clone repository from GitHub
.. git clone https://github.com/mansurali901/kafka-kubernetes.git
.. cd kafka-kubernetes
- Create Presistent Volumes for each pod with below order first we need to create presistent volume and then need to create presistent volume claim.

> Kafka PV, PVC
> zookeeper PV, PVC
> kafka-connect PV, PVC

-- Creating PVs for Kafka
cd kafka-volume-scripts/ 
.. kubectl create -f kafka-0-volume.yaml
.. kubectl create -f kafka-0-volume.yaml

- Do above steps for all three replicas
.. cd zookeeper-volume-scripts/
.. kubectl create -f zookeper-0-volume.yaml 
.. kubectl create -f zookeper-0-pvc.yaml

- Do this for Kafka connect as well
.. cd connect-volume-scripts/
.. kubectl create -f connect-0-volume.yaml
.. kubectl create -f connect-0-pvc.yaml

- Check the PVs and PVC status
==============================================================================================================================
.. root@ip-10-0-0-156:~/kafka# kubectl get pv 
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   REASON   AGE
connect0         10Gi       RWO            Retain           Bound    default/connect-connect-0   connect0                150m
connect1         10Gi       RWO            Retain           Bound    default/connect-connect-1   connect1                144m
connect2         10Gi       RWO            Retain           Bound    default/connect-connect-2   connect2                143m
kafka0           10Gi       RWO            Retain           Bound    default/kafka-kafka-0       kafka0                  3h1m
kafka1           10Gi       RWO            Retain           Bound    default/kafka-kafka-1       kafka1                  177m
kafka2           10Gi       RWO            Retain           Bound    default/kafka-kafka-2       kafka2                  168m
zk0              10Gi       RWO            Retain           Bound    default/zookeeper-zk-0      zk0                     163m
zk1              10Gi       RWO            Retain           Bound    default/zookeeper-zk-1      zk1                     159m
zk2              10Gi       RWO            Retain           Bound    default/zookeeper-zk-2      zk2                     156m
==============================================================================================================================
root@ip-10-0-0-156:~/kafka# kubectl get pvc
NAME                STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
connect-connect-0   Bound    connect0         10Gi       RWO            connect0       150m
connect-connect-1   Bound    connect1         10Gi       RWO            connect1       145m
connect-connect-2   Bound    connect2         10Gi       RWO            connect2       144m
kafka-kafka-0       Bound    kafka0           10Gi       RWO            kafka0         3h
kafka-kafka-1       Bound    kafka1           10Gi       RWO            kafka1         176m
kafka-kafka-2       Bound    kafka2           10Gi       RWO            kafka2         168m
zookeeper-zk-0      Bound    zk0              10Gi       RWO            zk0            163m
zookeeper-zk-1      Bound    zk1              10Gi       RWO            zk1            158m
zookeeper-zk-2      Bound    zk2              10Gi       RWO            zk2            155m
root@ip-10-0-0-156:~/kafka# 
===============================================================================================================================


# Now Deploy Kafka PODs to kubernetes Cluster
.. kubectl create -f deployment.yaml

- Check the deployment status
==============================================================================================================================
root@ip-10-0-0-156:~/kafka# kubectl get all
NAME              READY   STATUS    RESTARTS   AGE
pod/connect-0     0/1     Error     0          63s
pod/connect-1     1/1     Running   1          28m
pod/connect-2     0/1     Pending   0          35s
pod/kafka-0       1/1     Running   0          130m
pod/kafka-1       1/1     Running   0          27m
pod/kafka-2       0/1     Pending   0          1s
pod/task-pv-pod   1/1     Running   0          3h57m
pod/zk-0          1/1     Running   0          130m
pod/zk-1          1/1     Running   0          130m
pod/zk-2          0/1     Pending   0          130m

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/connect-rest   ClusterIP   10.97.96.182     <none>        8083/TCP            130m
service/kafka-broker   ClusterIP   10.101.127.73    <none>        9092/TCP            130m
service/kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP             6d3h
service/zk-cs          ClusterIP   10.109.107.152   <none>        2181/TCP            130m
service/zk-hs          ClusterIP   None             <none>        2888/TCP,3888/TCP   130m

NAME                       DESIRED   CURRENT   AGE
statefulset.apps/connect   3         3         130m
statefulset.apps/kafka     3         3         130m
statefulset.apps/zk        3         3         130m

==============================================================================================================================

# Issues (Name resolution issues)
==============================================================================================================================
% ERROR: Local: Host resolution failure: kafka-broker.default.svc.cluster.local:9092/1001: Failed to resolve 'kafka-broker.default.svc.cluster.local:9092': Temporary failure in name resolution
==============================================================================================================================

# Resolution 
Get the Kafka-broker Cluster-IP and map in hosts file 
service/kafka-broker   ClusterIP   10.101.127.73    <none>        9092/TCP            130m
  
.. vim /etc/hosts
10.101.127.73   kafka-broker.default.svc.cluster.local

Save the host file and test again 
