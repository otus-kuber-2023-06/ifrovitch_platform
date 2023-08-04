Домашнее задание. Шаблонизация манифестов Kubernetes

Кластер настроен в YC через kubespray

helm version
version.BuildInfo{Version:"v3.5.3", GitCommit:"041ce5a2c17a58be0fcd5f5e16fb3e7e95fea622", GitTreeState:"dirty", GoVersion:"go1.15.8"}

helm repo add stable https://charts.helm.sh/stable
"stable" has been added to your repositories

λ helm repo list
NAME    URL
stable  https://charts.helm.sh/stable

sudo kubectl create ns nginx-ingress
namespace/nginx-ingress created

helm upgrade --install nginx-ingress stable/nginx-ingress --wait  --namespace=nginx-ingress  --version=1.41.3 --dry-run

helm upgrade --install nginx-ingress stable/nginx-ingress --wait  --namespace=nginx-ingress  --version=1.41.3

kubectl --namespace nginx-ingress get services -o wide -w nginx-ingress-controller
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE    SELECTOR
nginx-ingress-controller   LoadBalancer   10.233.52.119   <pending>     80:30505/TCP,443:32236/TCP   101s   app.kubernetes.io/component=controller,app=nginx-ingress,release=nginx-ingress

WARNING: This chart is deprecated
Error: timed out waiting for the condition

helm list --namespace nginx-ingress
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS  CHART                   APP VERSION
nginx-ingress   nginx-ingress   1               2023-07-27 20:22:38.704346841 +0000 UTC failed  nginx-ingress-1.41.3    v0.34.1

helm uninstall nginx-ingress --namespace nginx-ingress

В итоге поставил так
https://kubernetes.github.io/ingress-nginx/deploy/

helm upgrade --install ingress-nginx ingress-nginx      --repo https://kubernetes.github.io/ingress-nginx      --namespace nginx-ingress

kubectl get all -n nginx-ingress
NAME                                            READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-5ff6bb675f-q762l   1/1     Running   0          8m21s

NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.233.29.12   <pending>     80:30625/TCP,443:30082/TCP   8m21s
service/ingress-nginx-controller-admission   ClusterIP      10.233.37.98   <none>        443/TCP                      8m21s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           8m21s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-5ff6bb675f   1         1         1       8m21s

kubectl edit configmap -n kube-system kube-proxy
kubectl edit configmap -n kube-system kube-proxy
kubectl get configmap -n kube-system kube-proxy -o yaml |grep strictARP

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml

sudo kubectl get all -n metallb-system

add ippool.yaml
sudo kubectl apply -f ippool.yaml
ipaddresspool.metallb.io/ip-pool created

sudo kubectl get all -n nginx-ingress
NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.233.19.107   10.128.0.1    80:31271/TCP,443:30934/TCP   



helm repo add jetstack https://charts.jetstack.io

"jetstack" has been added to your repositories

sudo kubectl create ns cert-manager
namespace/cert-manager created

helm upgrade --install cert-manager jetstack/cert-manager --wait --namespace=cert-manager --version v1.12.3 --set installCRDs=true

kubectl get all -n cert-manager
NAME                                           READY   STATUS    RESTARTS   AGE
pod/cert-manager-65dfbdf7d6-gfw2g              1/1     Running   0          3m14s
pod/cert-manager-cainjector-79f5dbffcf-6vmwd   1/1     Running   0          3m14s
pod/cert-manager-webhook-77b984cc67-z4wqg      1/1     Running   0          3m14s

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/cert-manager           ClusterIP   10.233.12.3     <none>        9402/TCP   3m14s
service/cert-manager-webhook   ClusterIP   10.233.30.202   <none>        443/TCP    3m14s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1/1     1            1           3m14s
deployment.apps/cert-manager-cainjector   1/1     1            1           3m14s
deployment.apps/cert-manager-webhook      1/1     1            1           3m14s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-65dfbdf7d6              1         1         1       3m14s
replicaset.apps/cert-manager-cainjector-79f5dbffcf   1         1         1       3m14s
replicaset.apps/cert-manager-webhook-77b984cc67      1         1         1       3m14s


*cert-managercert-manager Самостоятельное задание
Добавил манифест
cm-issuer.yaml

 sudo kubectl apply -f cm-issuer.yaml
clusterissuer.cert-manager.io/letsencrypt-production created

kubectl get ClusterIssuer letsencrypt-production -o yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"cert-manager.io/v1","kind":"ClusterIssuer","metadata":{"annotations":{},"name":"letsencrypt-production"},"spec":{"acme":{"email":"travis@io.com","privateKeySecretRef":{"name":"letsencrypt-production"},"server":"https://acme-v02.api.letsencrypt.org/directory","solvers":[{"http01":{"ingress":{"class":"nginx"}}}]}}}
  creationTimestamp: "2023-07-27T20:54:32Z"
  generation: 1
  name: letsencrypt-production
  resourceVersion: "217159"
  uid: fd82aad4-d1c8-4d3f-b8bf-9a49b88a46c7
spec:
  acme:
    email: travis@io.com
    preferredChain: ""
    privateKeySecretRef:
      name: letsencrypt-production
    server: https://acme-v02.api.letsencrypt.org/directory
    solvers:
    - http01:
        ingress:
          class: nginx
status:
  acme:
    lastPrivateKeyHash: FDp7B8ycEL0LFkRD73VWcl7NDoDVqeM/oT2c0cVLvQ8=
    lastRegisteredEmail: travis@io.com
    uri: https://acme-v02.api.letsencrypt.org/acme/acct/1228345976
  conditions:
  - lastTransitionTime: "2023-07-27T20:54:33Z"
    message: The ACME account was registered with the ACME server
    observedGeneration: 1
    reason: ACMEAccountRegistered
    status: "True"
    type: Ready



chartmuseum
add values.yaml

sudo kubectl create ns chartmuseum
namespace/chartmuseum created

helm upgrade --install chartmuseum stable/chartmuseum --wait \
 --namespace=chartmuseum \
 --version=2.13.2 \
 -f values.yaml

 Ошибки
 Ставил 
 helm upgrade --install chartmuseum chartmuseum/chartmuseum --wait      --namespace=chartmuseum --version=3.9.0 -f values.yaml

helm ls -n chartmuseum
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
chartmuseum     chartmuseum     2               2023-07-27 22:54:57.090370161 +0000 UTC deployed        chartmuseum-3.9.0       0.15.0

sudo kubectl get secrets -n chartmuseum
NAME                                TYPE                 DATA   AGE
chartmuseum                         Opaque               0      4m27s
sh.helm.release.v1.chartmuseum.v1   helm.sh/release.v1   1      4m27s
sh.helm.release.v1.chartmuseum.v2   helm.sh/release.v1   1      3m7s

sudo kubectl edit service/chartmuseum  -n chartmuseum
type: LoadBalancer

elinks https://chartmuseum.10.128.0.1.nip.io

 Welcome to ChartMuseum!

   If you see this page, the ChartMuseum web server is successfully installed and working.

   For online documentation and support please refer to the GitHub project.

   Thank you for using ChartMuseum.


chartmuseum | Задание со �
helm create mychartname
cd mychartname/
helm lint
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed

helm package .
Successfully packaged chart and saved it to: /home/sop/mychartname/mychartname-0.1.0.tgz

helm repo add my-chartmuseum https://chartmuseum.10.128.0.1.nip.io

helm cm-push  mychartname-0.1.0.tgz my-chartmuseum 
helm repo update

*harbor

kubectl create ns harbor
helm upgrade --install harbor harbor/harbor --version=1.12.2  -n=harbor --wait -f harbor/values.yaml --dry-run





