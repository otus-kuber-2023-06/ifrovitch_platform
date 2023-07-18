# ifrovitch_platform

Создание отдельной ветки kubernetes-intro
Создана структура репозитория
Добавлен frontend-pod-healthy.yaml
Добавлен web-pod.yaml
/kubernetes-intro/web и в ней структура
pod frontend ERROR - из-за отсутствия заданной переменной.

Разберитесь почему все pod в namespace kube-system восстановились после удаления
Некоторые поды в ns kube-system создаются control plane как static pods
они создаются напрямую kubelet из /etc/kubernetes/manifests
 если такой под удалить, kubelet заметит, что содержимое  /etc/kubernetes/manifests не соответствует действительности, и пересоздаст его
Отследеить можно через `# journalctl -u kubelet` изнутри minicube. Обыкновенные поды мониторятся controller-manager, и воссоздаются (при необходимости) kubelet`ом.


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












