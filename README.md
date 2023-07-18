# ifrovitch_platform
HW 1
Добавлены файлы
Автопроверка
MR

HW2
Создание отдельной ветки kubernetes-intro
Создана структура репозитория
Добавлен frontend-pod-healthy.yaml
Push образ в dockerhub
Добавлен web-pod.yaml
/kubernetes-intro/web и в ней структура
pod frontend ERROR - из-за отсутствия заданной переменной.


Разберитесь почему все pod в namespace kube-system восстановились после удаления
Некоторые поды в ns kube-system создаются control plane как static pods
они создаются напрямую kubelet из /etc/kubernetes/manifests
 если такой под удалить, kubelet заметит, что содержимое  /etc/kubernetes/manifests не соответствует действительности, и пересоздаст его
Отследеить можно через `# journalctl -u kubelet` изнутри minicube. Обыкновенные поды мониторятся controller-manager, и воссоздаются (при необходимости) kubelet`ом.

HW3

ДЗ Механика запуска и взаимодействия контейнеров в Kubernetes
1. установлен кластер через kubespray
sop@mas2:~$ ^C
sop@mas2:~$ sudo kubectl get nodes
NAME    STATUS   ROLES           AGE   VERSION
mas1    Ready    control-plane   14m   v1.26.6
mas2    Ready    control-plane   14m   v1.26.6
mas3    Ready    control-plane   13m   v1.26.6
work1   Ready    <none>          12m   v1.26.6
work2   Ready    <none>          12m   v1.26.6
work3   Ready    <none>          12m   v1.26.6

2. Ошибка при создании frontend
kubectl apply -f frontend-replicaset.yaml
Необходим селектор
Добавлено
----
  selector:
    matchLabels:
      app: frontend
----

3. Replicas
image: jupelok/hipster-frontend:v0.0.1
kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-bcbrx   1/1     Running   0          26s


kubectl get rs
NAME                        DESIRED   CURRENT   READY   AGE
frontend                    1         1         1       85s

Увеличение реплик
kubectl scale replicaset frontend --replicas=3
replicaset.apps/frontend scaled

kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-bcbrx   1/1     Running   0          2m2s
frontend-d2ddr   1/1     Running   0          11s
frontend-jqhdm   1/1     Running   0          11s

kubectl get rs frontend
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       3m7s

kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
NAME             READY   STATUS        RESTARTS   AGE
frontend-bcbrx   1/1     Terminating   0          3m38s
frontend-d2ddr   1/1     Running       0          107s
frontend-jqhdm   1/1     Running       0          107s
frontend-d2ddr   1/1     Terminating   0          107s
frontend-pnrt6   0/1     Pending       0          0s
frontend-jqhdm   1/1     Terminating   0          107s
frontend-pnrt6   0/1     Pending       0          0s
frontend-w9z8b   0/1     Pending       0          0s
frontend-w9z8b   0/1     Pending       0          0s
frontend-pnrt6   0/1     ContainerCreating   0          0s
frontend-w9z8b   0/1     ContainerCreating   0          0s
frontend-fc7c2   0/1     Pending             0          1s
frontend-fc7c2   0/1     Pending             0          1s
frontend-fc7c2   0/1     ContainerCreating   0          1s
frontend-d2ddr   0/1     Terminating         0          109s
frontend-d2ddr   0/1     Terminating         0          109s
frontend-d2ddr   0/1     Terminating         0          109s
frontend-jqhdm   0/1     Terminating         0          109s
frontend-jqhdm   0/1     Terminating         0          109s
frontend-jqhdm   0/1     Terminating         0          109s
frontend-bcbrx   0/1     Terminating         0          3m41s
frontend-bcbrx   0/1     Terminating         0          3m41s
frontend-bcbrx   0/1     Terminating         0          3m41s
frontend-pnrt6   1/1     Running             0          3s
frontend-w9z8b   1/1     Running             0          4s
frontend-fc7c2   1/1     Running             0          5s

kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-fc7c2   1/1     Running   0          73s
frontend-pnrt6   1/1     Running   0          73s
frontend-w9z8b   1/1     Running   0          73s

Применяю rs с 1 репликой
kubectl apply -f frontend-replicaset.yaml
replicaset.apps/frontend configured

kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-fc7c2   1/1     Running   0          2m

kubectl get rs
NAME                        DESIRED   CURRENT   READY   AGE
frontend                    1         1         1       6m43s

Внес исправления в манифест
image: jupelok/hipster-frontend:v0.0.2

kubectl apply -f frontend-replicaset.yaml
replicaset.apps/frontend configured

 kubectl get pods -l app=frontend -w
NAME             READY   STATUS    RESTARTS   AGE
frontend-fc7c2   1/1     Running   0          11m

kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
jupelok/hipster-frontend:v0.0.2
Новый имидж


kubectl delete pods -l app=frontend
kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
jupelok/hipster-frontend:v0.0.2

ReplicaSet гарантирует только факт заданного числа запущенных экземпляров подов в кластере Kubernetes в момент времени. Т.о. ReplicaSet не перезапускает поды при обновлении спецификации пода, в отличие от Deployment.





4. Deployment
image: jupelok/hipster-paymentservice:0.0.1

kubectl apply -f paymentservice-replicaset.yaml
replicaset.apps/paymentservice created
kubectl apply -f paymentservice-deployment.yaml
deployment.apps/paymentservice created


 kubectl get rs
 paymentservice   1         1         1       10s

kubectl get deployments.apps
paymentservice   1/1     1            1           11s

image: jupelok/hipster-paymentservice:0.0.2
kubectl apply -f paymentservice-deployment.yaml
deployment.apps/paymentservice configured

kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
NAME                              READY   STATUS    RESTARTS   AGE
paymentservice-58867c4d8d-8686q   1/1     Running   0          3m
paymentservice-58867c4d8d-c9flt   1/1     Running   0          3m
paymentservice-58867c4d8d-jj45k   1/1     Running   0          3m
paymentservice-5f757978f5-6wslc   0/1     Pending   0          0s
paymentservice-5f757978f5-6wslc   0/1     Pending   0          0s
paymentservice-5f757978f5-6wslc   0/1     ContainerCreating   0          0s
paymentservice-5f757978f5-6wslc   1/1     Running             0          4s
paymentservice-58867c4d8d-c9flt   1/1     Terminating         0          3m4s
paymentservice-5f757978f5-thktf   0/1     Pending             0          0s
paymentservice-5f757978f5-thktf   0/1     Pending             0          0s
paymentservice-5f757978f5-thktf   0/1     ContainerCreating   0          0s
paymentservice-5f757978f5-thktf   1/1     Running             0          4s
paymentservice-58867c4d8d-8686q   1/1     Terminating         0          3m8s
paymentservice-5f757978f5-n4b8l   0/1     Pending             0          0s
paymentservice-5f757978f5-n4b8l   0/1     Pending             0          0s
paymentservice-5f757978f5-n4b8l   0/1     ContainerCreating   0          0s
paymentservice-5f757978f5-n4b8l   1/1     Running             0          4s
paymentservice-58867c4d8d-jj45k   1/1     Terminating         0          3m12s
paymentservice-58867c4d8d-c9flt   0/1     Terminating         0          3m35s




5. Rollback


 kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l app=paymentservice -w
NAME                        DESIRED   CURRENT   READY   AGE
paymentservice              0         0         0       19m
paymentservice-85796c4dcb   1         1         1       3m45s
paymentservice              0         0         0       19m
paymentservice              1         0         0       19m
paymentservice              1         0         0       19m
paymentservice              1         1         0       19m
paymentservice              1         1         1       19m
paymentservice-85796c4dcb   0         1         1       3m47s
paymentservice-85796c4dcb   0         1         1       3m47s
paymentservice-85796c4dcb   0         0         0       3m47s

6.

Аналог blue-green:
. Развертывание трех новых pod
. Удаление трех старых pod
• Reverse Rolling Update:
. Удаление одного старого pod
. Создание одного нового pod

maxSurge это максимальное количество новых модулей, которые будут созданы за один раз, а maxUnavailable — это максимальное количество старых модулей, которые будут удалены за один раз
Значит в bg
      maxSurge: 3
      maxUnavailable: 3
Reverse
      maxSurge: 0
      maxUnavailable: 1



7. Probes
frontend-788b5b7795-7pf65         1/1     Running            0               3m26s
frontend-788b5b7795-b9ntj         1/1     Running            0               3m16s
frontend-788b5b7795-ggd2f         1/1     Running            0               3m47s
describe
Readiness:      http-get http://:8080/_healthz delay=10s timeout=1s period=10s #success=1 #failure=3

Events: none

После замены /_healthz на /_health

kubectl edit deployments.apps frontend
Warning  Unhealthy  8s (x6 over 58s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404

kubectl rollout status deployment/frontend
Waiting for deployment "frontend" rollout to finish: 1 out of 3 new replicas have been updated...

Вернул на /_healthz

kubectl apply -f frontend-deployment.yaml
Waiting for deployment "frontend" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "frontend" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "frontend" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "frontend" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "frontend" rollout to finish: 1 old replicas are pending termination...
deployment "frontend" successfully rolled out

8. Daemonset

curl localhost:9100/metrics | less
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.68954778189e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 7.34531584e+08
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes -1

9.
Добавление на мастер
Добавить в daemonset строки
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule

https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/



HW kubernetes-networks

1. Добавление проверок Pod
Add block to /web-pod.yml
---
readinessProbe: # Добавим проверку готовности
      httpGet: # веб-сервера отдавать
        path: /index.html # контент
        port: 80
---
kubectl apply -f web-pod.yaml
pod/web created

kubectl get pods | grep web
web                    1/1     Running   0          101s

 kubectl describe pod/web

Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True

 Warning  Unhealthy  21s (x18 over 2m41s)  kubelet            Readiness probe failed: Get "http://10.244.0.148:80/index.html": dial tcp 10.244.0.148:80: connect: connection refused


add block
---
  livenessProbe:
      tcpSocket: { port: 8000 }
---

kubectl apply -f web-pod.yaml
pod/web created

kubectl describe pod/web | grep Liveness
    Liveness:       tcp-socket :8000 delay=0s timeout=1s period=10s #success=1 #failure=3


Почему следующая конфигурация валидна, но не имеет смысла?
. Бывают ли ситуации, когда она все-таки имеет смысл?
livenessProbe:
exec:
command:
- 'sh'
- '-c'
- 'ps aux | grep my_web_server_process'

В ответ всегда 0, из-за процесса grep, смысла нет
ps aux | grep my_web_server_process
sop      1353390  0.0  0.1   6612  2180 pts/2    S+   11:43   0:00 grep --color=auto my_web_server_process
echo $?
0

Только в случае ps aux | grep my_web_server_process | grep -v grep, если из вывода убрать grep. Мониторинг начального уровня с метрикой, заущен ли какой-то процесс


2 Deployments
kubectl delete pod/web --grace-period=0 --force
kubectl apply -f web-deploy.yaml
kubectl describe deployment web

Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   web-549d6ddbc9 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  7s    deployment-controller  Scaled up replica set web-549d6ddbc9 to 3


порт 8000 3 replicas

Name:                   web
Namespace:              default
CreationTimestamp:      Mon, 17 Jul 2023 11:55:57 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=web
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web
  Init Containers:
   init-web:
    Image:      busybox:1.34.1
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      wget -O- https://tinyurl.com/otus-k8s-intro | sh
    Environment:  <none>
    Mounts:
      /app from app (rw)
  Containers:
   web:
    Image:        jupelok/otus:0.0.1
    Port:         <none>
    Host Port:    <none>
    Liveness:     tcp-socket :8000 delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:    http-get http://:8000/index.html delay=0s timeout=1s period=10s #success=1 #failure=3
    Startup:      http-get http://:8000/ delay=0s timeout=1s period=10s #success=1 #failure=30
    Environment:  <none>
    Mounts:
      /app from app (rw)
  Volumes:
   app:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   web-8cf67669f (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  11s   deployment-controller  Scaled up replica set web-8cf67669f to 3



kubectl get pods
web-8cf67669f-sz722    1/1     Running   0          16s
web-8cf67669f-vmqrx    1/1     Running   0          16s
web-8cf67669f-zd72s    1/1     Running   0          16s

 kubectl get deploy
 web              3/3     3            3           20s


2.
Add block
----
strategy:
type: RollingUpdate
rollingUpdate:
maxUnavailable: 0
maxSurge: 100%
----
maxUnavailable: 0
maxSurge: 0

kubectl get events --watch
kubespy trace deploy web

kubectl apply -f web-deploy.yaml
The Deployment "web" is invalid: spec.strategy.rollingUpdate.maxUnavailable: Invalid value: intstr.IntOrString{Type:0, IntVal:0, StrVal:""}: may not be 0 when `maxSurge` is 0

maxUnavailable: 100%
maxSurge: 100%

0s          Normal    Killing                   pod/web-7cbb7dc674-486g2    Stopping container web
0s          Normal    Scheduled                 pod/web-5cc4f55d79-v797c    Successfully assigned default/web-5cc4f55d79-v797c to minikube
0s          Normal    SuccessfulCreate          replicaset/web-5cc4f55d79   Created pod: web-5cc4f55d79-v797c
0s          Normal    Killing                   pod/web-7cbb7dc674-nwfv2    Stopping container web
0s          Normal    SuccessfulDelete          replicaset/web-7cbb7dc674   Deleted pod: web-7cbb7dc674-brbpd
0s          Normal    Scheduled                 pod/web-5cc4f55d79-g6cj7    Successfully assigned default/web-5cc4f55d79-g6cj7 to minikube


      maxUnavailable: 0
      maxSurge: 100%

5m20s       Normal    ScalingReplicaSet         deployment/web              Scaled up replica set web-5cc4f55d79 to 3
5m20s       Normal    ScalingReplicaSet         deployment/web              Scaled down replica set web-7cbb7dc674 to 0
0s          Normal    ScalingReplicaSet         deployment/web              Scaled up replica set web-7cbb7dc674 to 3
0s          Normal    SuccessfulCreate          replicaset/web-7cbb7dc674   Created pod: web-7cbb7dc674-ds5rs
0s          Normal    SuccessfulCreate          replicaset/web-7cbb7dc674   Created pod: web-7cbb7dc674-47wq8
0s          Normal    Scheduled                 pod/web-7cbb7dc674-ds5rs    Successfully assigned default/web-7cbb7dc674-ds5rs to minikube
0s          Normal    SuccessfulCreate          replicaset/web-7cbb7dc674   Created pod: web-7cbb7dc674-shh8t
0s          Normal    Scheduled                 pod/web-7cbb7dc674-shh8t    Successfully assigned default/web-7cbb7dc674-shh8t to minikube
0s          Normal    Scheduled                 pod/web-7cbb7dc674-47wq8    Successfully assigned default/web-7cbb7dc674-47wq8 to minikube


3
kubectl apply -f web-svc-cip.yaml
kubectl get services | grep web
web-svc-cip               ClusterIP      10.107.242.189   <none>        80/TCP         74s

 minikube ssh
 curl http://10.107.242.189/index.html

 export PAYMENTSERVICE_EXTERNAL_SERVICE_PORT_HTTP='80'
export PAYMENTSERVICE_PORT='tcp://10.105.93.31:80'
export PAYMENTSERVICE_PORT_80_TCP='tcp://10.105.93.31:80'
export PAYMENTSERVICE_PORT_80_TCP_ADDR='10.105.93.31'
export PAYMENTSERVICE_PORT_80_TCP_PORT='80'
export PAYMENTSERVICE_PORT_80_TCP_PROTO='tcp'
export PAYMENTSERVICE_SERVICE_HOST='10.105.93.31'
export PAYMENTSERVICE_SERVICE_PORT='80'
export PAYMENTSERVICE_SERVICE_PORT_HTTP='80'
export PWD='/'
export SHLVL='2'</pre>
<h3>Memory info</h3>
<pre>              total        used        free      shared  buff/cache   available
Mem:           1963         887         135          73         942         813
Swap:             0           0           0</pre>
<h3>DNS resolvers info</h3>
<pre>nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local ru-central1.internal auto.internal
options ndots:5</pre>
<h3>Static hosts info</h3>
<pre># Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.244.0.161    web-8cf67669f-sz722</pre>
</body>
</html>


 ping 10.107.242.189
PING 10.107.242.189 (10.107.242.189) 56(84) bytes of data.


arp -an
ip addr show
IP none

iptables --list -nv -t nat | grep 10.107.242.189
    3   180 KUBE-SVC-6CZTMAROCN3AQODZ  tcp  --  *      *       0.0.0.0/0            10.107.242.189       /* default/web-svc-cip cluster IP */ tcp dpt:80
    3   180 KUBE-MARK-MASQ  tcp  --  *      *      !10.244.0.0           10.107.242.189       /* default/web-svc-cip cluster IP */ tcp dpt:80

kubectl --namespace kube-system edit
configmap/kube-proxy

add
---
ipvs:
strictARP: true
mode: "ipvs"
---

 minikube ssh

 touch /tmp/iptables.cleanup
 add
*nat
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
COMMIT
*filter
COMMIT
*mangle
COMMIT

  iptables-restore /tmp/iptables.cleanup

  iptables --list -nv -t nat

  toolbox none

apt install install ipvsadm 

ipvsadm --list -n

ping clusterIP OK

4 Установка MetalLB
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13/config/manifests/metallb-native.yaml

kubectl --namespace metallb-system get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/controller-9d798dd5b-mrlfb   1/1     Running   0          5m29s
pod/speaker-j7m5n                1/1     Running   0          28m

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/webhook-service   ClusterIP   10.111.1.98   <none>        443/TCP   28m

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   28m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           28m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-599869d975   0         0         0       23m
replicaset.apps/controller-69f9899756   0         0         0       12m
replicaset.apps/controller-9d798dd5b    1         1         1       5m30s
replicaset.apps/controller-d7b4f4dc7    0         0         0       23m

 kubectl apply -f web-svc-lb.yaml

 kubectl describe svc web-svc-lb
 Normal   IPAllocated       10s   metallb-controller  Assigned IP ["172.17.255.1"]
kubectl --namespace metallb-system logs pod/controller-9d798dd5b-mrlfb
 {"caller":"service.go:142","event":"ipAllocated","ip":["172.17.255.1"],"level":"info","msg":"IP address assigned by controller

 ip addr show eth0
 192.168.49.2

 sudo ip route add  172.17.255.0/24 via 192.168.49.2

elinks 172.17.255.1

5. Задание со � | DNS через MetalLB
kubectl apply -f svc-lb.yaml

service/svc-tcp created
service/svc-udp created

 kubectl get service -n kube-system

svc-tcp    LoadBalancer   10.106.160.227   172.17.255.10   53:31196/TCP             11s
svc-udp    LoadBalancer   10.111.2.246     172.17.255.10   53:31365/UDP             11s

nslookup web-svc-lb.default.svc.cluster.local 172.17.255.10
Server:         172.17.255.10
Address:        172.17.255.10#53

Name:   web-svc-lb.default.svc.cluster.local
Address: 10.104.211.41

6. Ingress

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master
/deploy/static/provider/baremetal/deploy.yaml

serviceaccount/ingress-nginx configured
serviceaccount/ingress-nginx-admission configured
role.rbac.authorization.k8s.io/ingress-nginx configured
role.rbac.authorization.k8s.io/ingress-nginx-admission configured
clusterrole.rbac.authorization.k8s.io/ingress-nginx configured
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission configured
rolebinding.rbac.authorization.k8s.io/ingress-nginx configured
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission configured
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx configured
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission configured
configmap/ingress-nginx-controller configured
service/ingress-nginx-controller configured
service/ingress-nginx-controller-admission configured
deployment.apps/ingress-nginx-controller configured
ingressclass.networking.k8s.io/nginx configured

kubectl apply -f nginx-lb.yaml

kubectl get services
ingress-nginx    ingress-nginx                        LoadBalancer   10.109.151.189   172.17.255.4    80:31862/TCP,443:32219/TCP   42s

curl 172.17.255.4
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>

add web-svc-headless.yaml

kubectl apply -f web-svc-headless.yaml

kubectl get services  | grep web-svc
web-svc                   ClusterIP      None             <none>         80/TCP         49s

add web-ingress.yaml

kubectl apply -f web-ingress.yaml
ingress.networking.k8s.io/web created

 kubectl describe ingress/web

  curl http://172.17.255.4/web/index.html
<html>
<head/>
<body>
<!-- IMAGE BEGINS HERE -->
<font size="-3">

7. Задания со � | Ingress для Dashboard

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

add dashboard.yaml

kubectl apply -f dashboard.yaml

kubectl describe ingress  dashboard -n kubernetes-dashboard

Name:             dashboard
Labels:           <none>
Namespace:        kubernetes-dashboard
Address:          192.168.49.2
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /dashboard   kubernetes-dashboard:443 (10.244.0.186:8443)
Annotations:  kubernetes.io/ingress.class: nginx
              nginx.ingress.kubernetes.io/backend-protocol: HTTPS
              nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age                   From                      Message
  ----    ------  ----                  ----                      -------
  Normal  Sync    4m43s (x3 over 8m2s)  nginx-ingress-controller  Scheduled for sync

  curl --insecure http://172.17.255.4/dashboard

  ---
  <!--
Copyright 2017 The Kubernetes Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
--><!DOCTYPE html><html lang="en" dir="ltr"><head>
  <meta charset="utf-8">
  <title>Kubernetes Dashboard</title>
  <link rel="icon" type="image/png" href="assets/images/kubernetes-logo.png">
  <meta name="viewport" content="width=device-width">
<style>html,body{height:100%;margin:0}*::-webkit-scrollbar{background:transparent;height:8px;width:8px}</style><link rel="stylesheet" href="styles.243e6d874431c8e8.css" media="print" onload="this.media='all'"><noscript><link rel="stylesheet" href="styles.243e6d874431c8e8.css"></noscript></head>

<body>
  <kd-root></kd-root>
<script src="runtime.134ad7745384bed8.js" type="module"></script><script src="polyfills.5c84b93f78682d4f.js" type="module"></script><script src="scripts.2c4f58d7c579cacb.js" defer></script><script src="en.main.3550e3edca7d0ed8.js" type="module"></script>


</body></html>
---

















