Домашнее задание. Шаблонизация манифестов Kubernetes

Кластер настроен в YC как Managed Service for Kubernetes


helm version
version.BuildInfo{Version:"v3.5.3", GitCommit:"041ce5a2c17a58be0fcd5f5e16fb3e7e95fea622", GitTreeState:"dirty", GoVersion:"go1.15.8"}

kubectl create ns ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
kubectl get all -n ingress-nginx
NAME                                          READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-78d54fbd-s5vk8   1/1     Running   0          3m17s

NAME                                         TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   172.16.225.178   158.160.104.163   80:30738/TCP,443:31742/TCP   3m17s
service/ingress-nginx-controller-admission   ClusterIP      172.16.212.155   <none>            443/TCP                      3m17s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           3m17s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-78d54fbd   1         1         1       3m17s




helm repo add jetstack https://charts.jetstack.io

"jetstack" has been added to your repositories

sudo kubectl create ns cert-manager
namespace/cert-manager created

helm upgrade --install cert-manager jetstack/cert-manager --wait --namespace=cert-manager --version v1.12.3 --set installCRDs=true

kubectl get all -n cert-manager
NAME                                           READY   STATUS    RESTARTS   AGE
pod/cert-manager-5674b9b755-mwl5s              1/1     Running   0          28s
pod/cert-manager-cainjector-557c547f54-ns8c6   1/1     Running   0          28s
pod/cert-manager-webhook-86868b95db-rj2m7      1/1     Running   0          28s

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/cert-manager           ClusterIP   172.16.141.208   <none>        9402/TCP   29s
service/cert-manager-webhook   ClusterIP   172.16.148.163   <none>        443/TCP    29s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1/1     1            1           28s
deployment.apps/cert-manager-cainjector   1/1     1            1           28s
deployment.apps/cert-manager-webhook      1/1     1            1           28s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-5674b9b755              1         1         1       28s
replicaset.apps/cert-manager-cainjector-557c547f54   1         1         1       28s
replicaset.apps/cert-manager-webhook-86868b95db      1         1         1       28s



*cert-managercert-manager Самостоятельное задание
Добавил манифест
cm-issuer.yaml
kubectl apply -f cm-issuer.yaml
clusterissuer.cert-manager.io/letsencrypt-production created

kubectl get ClusterIssuer letsencrypt-production -o yaml
kubectl get ClusterIssuer letsencrypt-production -o yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"cert-manager.io/v1","kind":"ClusterIssuer","metadata":{"annotations":{},"name":"letsencrypt-production"},"spec":{"acme":{"email":"travis@io.com","privateKeySecretRef":{"name":"letsencrypt-production"},"server":"https://acme-v02.api.letsencrypt.org/directory","solvers":[{"http01":{"ingress":{"class":"nginx"}}}]}}}
  creationTimestamp: "2023-07-30T14:43:30Z"
  generation: 1
  managedFields:
  - apiVersion: cert-manager.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
      f:spec:
        .: {}
        f:acme:
          .: {}
          f:email: {}
          f:privateKeySecretRef:
            .: {}
            f:name: {}
          f:server: {}
          f:solvers: {}
    manager: kubectl-client-side-apply
    operation: Update
    time: "2023-07-30T14:43:30Z"
  - apiVersion: cert-manager.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        .: {}
        f:acme:
          .: {}
          f:lastPrivateKeyHash: {}
          f:lastRegisteredEmail: {}
          f:uri: {}
        f:conditions:
          .: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastTransitionTime: {}
            f:message: {}
            f:observedGeneration: {}
            f:reason: {}
            f:status: {}
            f:type: {}
    manager: cert-manager-clusterissuers
    operation: Update
    subresource: status
    time: "2023-07-30T14:43:31Z"
  name: letsencrypt-production
  resourceVersion: "21803"
  uid: 8dcb4623-cb3f-4c1a-a844-a30c0c52d27a
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
    lastPrivateKeyHash: kCFw/XiViD6awmIVNIUqd8miwDaWWy/AAkp5icBg2Ow=
    lastRegisteredEmail: travis@io.com
    uri: https://acme-v02.api.letsencrypt.org/acme/acct/1232357896
  conditions:
  - lastTransitionTime: "2023-07-30T14:43:31Z"
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
 helm repo add chartmuseum https://chartmuseum.github.io/charts
 helm upgrade --install chartmuseum chartmuseum/chartmuseum --wait      --namespace=chartmuseum --version=3.9.0 -f values.yaml

helm ls -n chartmuseum
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
chartmuseum     chartmuseum     1               2023-07-30 17:48:51.8090925 +0300 MSK   deployed        chartmuseum-3.9.0       0.15.0

kubectl get secrets -n chartmuseum
NAME                                TYPE                 DATA   AGE
chartmuseum                         Opaque               0      74s
sh.helm.release.v1.chartmuseum.v1   helm.sh/release.v1   1      74s

kubectl get all -n chartmuseum
NAME                               READY   STATUS    RESTARTS   AGE
pod/chartmuseum-59b4dd9655-md2nc   1/1     Running   0          111s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/chartmuseum   ClusterIP   172.16.152.94   <none>        8080/TCP   113s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/chartmuseum   1/1     1            1           112s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/chartmuseum-59b4dd9655   1         1         1       112s

kubectl get certificate -n chartmuseum
NAME                                   READY   SECRET                                 AGE
chartmuseum.158.160.104.163.sslip.io   True    chartmuseum.158.160.104.163.sslip.io   32s

elinks chartmuseum.158.160.104.163.sslip.io 
---
                                                                  Welcome to ChartMuseum!

   If you see this page, the ChartMuseum web server is successfully installed and working.

   For online documentation and support please refer to the GitHub project.

   Thank you for using ChartMuseum.
---



chartmuseum | Задание со �
helm repo add bitnami https://charts.bitnami.com/bitnami
helm pull bitnami/gitea
kubectl get all -n chartmuseum
kubectl edit deployment.apps/chartmuseum -n chartmuseum
DISABLE_API to false
helm cm-push gitea-0.3.6.tgz my-chartmuseum
Pushing gitea-0.3.6.tgz to my-chartmuseum...
Done.
my-chartmuseum/gitea    0.3.6           1.19.4          Gitea is a lightweight code hosting
helm upgrade --install gitea my-chartmuseum/gitea --version=0.3.6


*harbor

kubectl create ns harbor
add values.yaml
helm upgrade --install harbor harbor/harbor --version=1.12.2  -n=harbor --wait -f harbor/values.yaml --dry-run


helm upgrade --install harbor harbor/harbor --version=1.12.2  -n=harbor --wait -f values.yaml

kubectl get certificates -n harbor
NAME         READY   SECRET       AGE
harbor-tls   True    harbor-tls   72s

NAME                                     READY   STATUS    RESTARTS       AGE
pod/harbor-core-6554db874-2gz59          1/1     Running   0              2m41s
pod/harbor-database-0                    1/1     Running   0              2m41s
pod/harbor-jobservice-7c9c8bcff7-v7pnk   1/1     Running   2 (2m1s ago)   2m41s
pod/harbor-portal-7f9dd585cc-pxn99       1/1     Running   0              2m41s
pod/harbor-redis-0                       1/1     Running   0              2m41s
pod/harbor-registry-7d7b7bbcd-z5qxf      2/2     Running   0              2m41s
pod/harbor-trivy-0                       1/1     Running   0              2m41s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/harbor-core         ClusterIP   172.16.149.107   <none>        80/TCP              2m41s
service/harbor-database     ClusterIP   172.16.171.121   <none>        5432/TCP            2m41s
service/harbor-jobservice   ClusterIP   172.16.152.169   <none>        80/TCP              2m41s
service/harbor-portal       ClusterIP   172.16.172.23    <none>        80/TCP              2m41s
service/harbor-redis        ClusterIP   172.16.131.215   <none>        6379/TCP            2m41s
service/harbor-registry     ClusterIP   172.16.197.210   <none>        5000/TCP,8080/TCP   2m41s
service/harbor-trivy        ClusterIP   172.16.170.24    <none>        8080/TCP            2m41s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/harbor-core         1/1     1            1           2m41s
deployment.apps/harbor-jobservice   1/1     1            1           2m41s
deployment.apps/harbor-portal       1/1     1            1           2m41s
deployment.apps/harbor-registry     1/1     1            1           2m41s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/harbor-core-6554db874          1         1         1       2m41s
replicaset.apps/harbor-jobservice-7c9c8bcff7   1         1         1       2m41s
replicaset.apps/harbor-portal-7f9dd585cc       1         1         1       2m41s
replicaset.apps/harbor-registry-7d7b7bbcd      1         1         1       2m41s

NAME                               READY   AGE
statefulset.apps/harbor-database   1/1     2m41s
statefulset.apps/harbor-redis      1/1     2m41s
statefulset.apps/harbor-trivy      1/1     2m41s

kubectl get secrets -n harbor -l owner=helm

NAME                           TYPE                 DATA   AGE
sh.helm.release.v1.harbor.v1   helm.sh/release.v1   1      3m49s


https://harbor.158.160.104.163.sslip.io/ открывается


Используем helmfile | Задание со �

scoop install helmfile
helm plugin install https://github.com/databus23/helm-diff
Installed plugin: diff

helmfile -e dev apply


Создаем свой helm chart
helm create kubernetes-templating/hipster-shop
kubectl create ns hipster-shop
rm all on templates and values
cp all-hipster-shop.yaml 

helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
Release "hipster-shop" does not exist. Installing it now.
NAME: hipster-shop
LAST DEPLOYED: Sun Jul 30 22:41:18 2023
NAMESPACE: hipster-shop
STATUS: deployed
REVISION: 1
TEST SUITE: None

helm create kubernetes-templating/frontend
rm all on templates and values
add ingress.yaml service.yaml deployment.yaml

helm delete frontend -n hipster-shop
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
Release "hipster-shop" has been upgraded. Happy Helming!
NAME: hipster-shop
LAST DEPLOYED: Sun Jul 30 22:48:42 2023
NAMESPACE: hipster-shop
STATUS: deployed
REVISION: 2
TEST SUITE: None

helm upgrade --install frontend kubernetes-templating/frontend --namespace hipster-shop


add values
---
image:
  tag: v0.8.0
replicas: 1
ports:
  containerPort: 8080
service:
  type: NodePort
  port: 80
  targetPort: 8080
  nodePort: 30001
  name: frontend
ingress:
  host: shopp.158.160.104.163.sslip.io
  backendPort: 80
deployments:
  name: frontend
---

add dep in Chart.yaml hipster-shop
λ ls -la kubernetes-templating/hipster-shop/charts/
total 4
drwxr-xr-x 1 sop 197121    0 июл 31 00:06 ./
drwxr-xr-x 1 sop 197121    0 июл 31 00:06 ../
-rw-r--r-- 1 sop 197121 1660 июл 31 00:06 frontend-0.1.0.tgz

helm dep update kubernetes-templating/hipster-shop
kubectl get all -n hipster-shop | grep frontend
pod/frontend-748c98cd89-xb2zg                1/1     Running            0          107s
service/frontend                NodePort    172.16.237.35    <none>        80:30001/TCP   108s
deployment.apps/frontend                1/1     1            1           107s
replicaset.apps/frontend-748c98cd89                1         1         1       107s

helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop --set frontend.service.NodePort=31234
wrong value

helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop --set frontend.service.nodePort=31234
good

Создаем свой helm chart | Задание со �

add requirements.yaml
delete block about service redis from ll-hipster-shop.yaml
add values.yaml
helm delete hipster-shop -n hipster-shop
helm dep update kubernetes-templating/hipster-shop
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop

Kubecfg

add files
├── paymentservice-deployment.yaml
├── paymentservice-service.yaml
├── shippingservice-deployment.yaml
└── shippingservice-service.yaml

add services.jsonnet


Kustomize | Самостоятельное задание

Service - # emailservice
add
emailservice-deployment.yaml
emailservice-service.yaml
kustomization.yaml

Установка в 2 окружения - test и prod
Папки hipster-shop-test и hipster-shop-prod

Удаление email из all-hipster-shop.yaml
Обновление релиза без email

Установка в тест
kubectl apply -k kubernetes-templating/kustomize/overrides/hipster-shop-test/
service/test-emailservice created
deployment.apps/test-emailservice created

service/test-emailservice             ClusterIP   172.16.189.102   <none>        5000/TCP       48s
deployment.apps/test-emailservice       1/1     1            1           48s

Установка в прод
kubectl create ns hipster-shop-prod
kubectl apply -k kubernetes-templating/kustomize/overrides/hipster-shop-prod
service/prod-emailservice created
deployment.apps/prod-emailservice created

λ kubectl get all -n hipster-shop-prod
NAME                                     READY   STATUS    RESTARTS   AGE
pod/prod-emailservice-58b7976894-9sp7t   1/1     Running   0          20s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/prod-emailservice   ClusterIP   172.16.154.209   <none>        5000/TCP   20s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prod-emailservice   1/1     1            1           20s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/prod-emailservice-58b7976894   1         1         1       20s
