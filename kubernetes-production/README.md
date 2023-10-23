kubernetes-production

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a



sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

sudo modprobe overlay
sudo modprobe br_netfilter
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
# Apply sysctl params without reboot
sudo sysctl --system
#Install and configure containerd
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update -y
sudo apt install -y containerd.io
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
#Start containerd
sudo systemctl restart containerd
sudo systemctl enable containerd

sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update && sudo apt-get -y install vim git curl wget kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00
sudo apt-mark hold kubelet kubeadm kubectl
sudo kubeadm config images pull --cri-socket /run/containerd/containerd.sock --kubernetes-version
v1.23.0

sudo systemctl enable containerd
sudo systemctl start containerd
sudo systemctl status containerd
rm /etc/containerd/config.toml
systemctl restart containerd

sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --upload-certs --kubernetes-version=v1.23.0 --ignore-preflight-errors=Mem --cri-socket /run/containerd/containerd.sock


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

kubeadm join 10.128.0.29:6443 --token 9hn78f.nhsw22jpl3adneq5 \
        --discovery-token-ca-cert-hash sha256:7732f390e148e418fd4fda84d22ab6e3fcd9ce92a4655dfdaec44b3cf6d6dc02


mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes
NAME     STATUS     ROLES                  AGE    VERSION
master   NotReady   control-plane,master   115s   v1.23.0

 kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-certhash sha256:<hash></hash></master-port></master-ip></token>

sudo rm /etc/containerd/config.toml
sudo systemctl restart containerd

sudo kubeadm join 10.128.0.29:6443 --token 9hn78f.nhsw22jpl3adneq5 --discovery-token-ca-cert-hash sha256:7732f390e148e418fd4fda84d22ab6e3fcd9ce92a4655dfdaec44b3cf6d6dc02


kubectl get nodes
NAME       STATUS     ROLES                  AGE     VERSION
master     Ready      control-plane,master   18m     v1.23.0
worker01   Ready      <none>                 2m59s   v1.23.0
worker02   NotReady   <none>                 2s      v1.23.0
worker03   Ready      <none>                 26s     v1.23.0


kubectl apply -f deployment.yaml
deployment.apps/nginx-deployment created

sudo apt update
sudo apt-cache madison kubeadm
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=1.24.0-00 
sudo apt-mark hold kubeadm
kubeadm upgrade plan
sudo kubeadm upgrade apply v1.24.0

 kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master     Ready    control-plane   75m   v1.23.0   10.128.0.29   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker01   Ready    <none>          60m   v1.23.0   10.128.0.35   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker02   Ready    <none>          57m   v1.23.0   10.128.0.6    <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker03   Ready    <none>          57m   v1.23.0   10.128.0.21   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24

 kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", GitCommit:"4ce5a8954017644c5420bae81d72b09b735c21f0", GitTreeState:"clean", BuildDate:"2022-05-03T13:44:24Z", GoVersion:"go1.18.1", Compiler:"gc", Platform:"linux/amd64"}

kubelet --version
Kubernetes v1.23.0

kubectl describe pod/kube-apiserver-master  -n kube-system
 Normal  Pulled   5m13s  kubelet  Container image "k8s.gcr.io/kube-apiserver:v1.24.0" already present on machine


 какая версия у мастер ноды? Почему?

Потому что kubelet --version
Kubernetes v1.23.0
На ноде отображается версия Kubernetes (версии kubelet и kube-proxy)

какая у kubelet?
kubelet --version
Kubernetes v1.23.0
Потому что обновил kubeadm

Какая версия у Api сервера?
v1.24.0

sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.24.0

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.24.0". Enjoy!

kubectl drain worker01
node/worker01 cordoned
error: unable to drain node "worker01" due to error:cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-flannel/kube-flannel-ds-rcs7v, kube-system/kube-proxy-hjcpl, continuing command...
There are pending nodes to be drained:
 worker01
cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-flannel/kube-flannel-ds-rcs7v, kube-system/kube-proxy-hjcpl

kubectl drain worker01 --ignore-daemonsets
node/worker01 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-rcs7v, kube-system/kube-proxy-hjcpl
evicting pod default/nginx-deployment-8c9c5f4-hf9d7
evicting pod default/nginx-deployment-8c9c5f4-5pbdn
pod/nginx-deployment-8c9c5f4-hf9d7 evicted
pod/nginx-deployment-8c9c5f4-5pbdn evicted
node/worker01 drained




sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=1.24.0-00 && sudo apt-mark hold kubeadm
sudo kubeadm upgrade node
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet=1.24.0-00
kubectl=1.24.0-00 && sudo apt-mark hold kubelet kubectl

Preparing to unpack .../kubelet_1.24.0-00_amd64.deb ...
Unpacking kubelet (1.24.0-00) over (1.23.0-00) ...
Setting up kubelet (1.24.0-00) ..

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl get nodes -o wide
NAME       STATUS                     ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master     Ready                      control-plane   85m   v1.23.0   10.128.0.29   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker01   Ready,SchedulingDisabled   <none>          70m   v1.24.0   10.128.0.35   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker02   Ready                      <none>          67m   v1.23.0   10.128.0.6    <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker03   Ready                      <none>          67m   v1.23.0   10.128.0.21   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24

Так как на worker-ноде обновили и kubelet (1.24.0-00), на ней отображена версия v1.24.0
Не уверен, что так задумано. Возможно ошибка в методичке, так как команды для мастер и воркер нод по обновлению компонентов различны.

kubectl uncordon worker01
node/worker01 uncordoned

На мастере
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet=1.24.0-00
=1.24.0-00 && sudo apt-mark hold kubelet kubectl

Preparing to unpack .../kubelet_1.24.0-00_amd64.deb ...
Unpacking kubelet (1.24.0-00) over (1.23.0-00) ...
Setting up kubelet (1.24.0-00) ...
sop@master:~$ kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master     Ready    control-plane   92m   v1.24.0   10.128.0.29   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker01   Ready    <none>          76m   v1.24.0   10.128.0.35   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker02   Ready    <none>          73m   v1.23.0   10.128.0.6    <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker03   Ready    <none>          74m   v1.23.0   10.128.0.21   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24

Т.о. мастер ноду обновли до новой версии


На оставшихся воркерах
kubectl drain worker02 --ignore-daemonsets
WARNING: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-jxb69, kube-system/kube-proxy-khcjk
evicting pod default/nginx-deployment-8c9c5f4-wr5lm
evicting pod default/nginx-deployment-8c9c5f4-l7j27
pod/nginx-deployment-8c9c5f4-l7j27 evicted
pod/nginx-deployment-8c9c5f4-wr5lm evicted
node/worker02 drained
kubectl get nodes -o wide
NAME       STATUS                     ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master     Ready                      control-plane   94m   v1.24.0   10.128.0.29   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker01   Ready                      <none>          79m   v1.24.0   10.128.0.35   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker02   Ready,SchedulingDisabled   <none>          76m   v1.23.0   10.128.0.6    <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
worker03   Ready                      <none>          77m   v1.23.0   10.128.0.21   <none>        Ubuntu 20.04.6 LTS   5.4.0-164-generic   containerd://1.6.24
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=1.24.0-00 && sudo apt-mark hold kubeadm
sudo kubeadm upgrade node
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet=1.24.0-00
kubectl=1.24.0-00 && sudo apt-mark hold kubelet kubectl
Preparing to unpack .../kubelet_1.24.0-00_amd64.deb ...
Unpacking kubelet (1.24.0-00) over (1.23.0-00) ...
Setting up kubelet (1.24.0-00) ...
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon worker02


kubectl drain worker03 --ignore-daemonsets
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=1.24.0-00 && sudo apt-mark hold kubeadm
sudo kubeadm upgrade node
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet=1.24.0-00
kubectl=1.24.0-00 && sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon worker03

 kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
master     Ready    control-plane   98m   v1.24.0
worker01   Ready    <none>          83m   v1.24.0
worker02   Ready    <none>          80m   v1.24.0
worker03   Ready    <none>          80m   v1.24.0

Задание со �

установлен кластер через kubespray

inventory/mycluster2/hosts.yaml
[all]
master01 ansIble_host=10.128.0.17 ip=10.128.0.17
master02 ansIble_host=10.128.0.22 ip=10.128.0.22
master03 ansIble_host=10.128.0.7 ip=10.128.0.7
work01 ansIble_host=10.128.0.26 ip=10.128.0.26
work02 ansIble_host=10.128.0.31 ip=10.128.0.31


[kube_control_plane]
master01
master02
master03

[etcd]
master01
master02
master03

[kube_node]
work01
work02

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr                                                                                            


ansible-playbook -i inventory/mycluster2/hosts.yaml --become --become-user=root cluster.yml

sudo kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
master01   Ready    control-plane   4m41s   v1.26.6
master02   Ready    control-plane   4m7s    v1.26.6
master03   Ready    control-plane   3m51s   v1.26.6
work01     Ready    <none>          2m50s   v1.26.6
work02     Ready    <none>          2m50s   v1.26.6

