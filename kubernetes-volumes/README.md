5   HW Volumes, Storages, StatefulSet
5.1
git checkout -b kubernetes-volumes
kubectl apply -f https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-02/Kuberenetes-volumes/minio-statefulset.yaml

statefulset.apps/minio created

Но в конфигурации возникли ошибки поэтому создал руками манифесты
sclass.yaml
create-pv.yaml
На ворекрах создал папки /data
И установил storageclass и pv
sudo kubectl edit pvc data-minio-0

 Добавил в spec
 ---
 spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-storage

---
Файлы прикладываю.

sudo kubectl get pods | grep minio
minio-0                    1/1     Running   0             7m21s

sudo kubectl get pvc
NAME           STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS    AGE
data-minio-0   Bound    pv-volume   10Gi       RWO            local-storage

5.2
Применение Headless Service
kubectl apply -f https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-02/Kuberenetes-volumes/minio-headless-service.yaml

kubectl get statefulsets
NAME    READY   AGE
minio   1/1     79m

sudo kubectl get pods | grep minio
minio-0                    1/1     Running   0             25m

sudo kubectl get pvc
NAME           STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS    AGE
data-minio-0   Bound    pv-volume   10Gi       RWO            local-storage   81m

sudo kubectl get pv
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS    REASON   AGE
pv-volume   10Gi       RWO            Retain           Bound    default/data-minio-0   local-storage            40m

Задание со *
kubectl delete statefulset minio
sudo kubectl apply -f minio-secret.yaml

sudo kubectl apply -f minio-statefulset.yaml

sudo kubectl get pods | grep minio
minio-0                    1/1     Running   0              2m36s

sudo kubectl get secrets
NAME           TYPE     DATA   AGE
minio-secret   Opaque   2      5m3s


5.3 Создание и использование
PersistentVolumeClaim в
Kubernetes(опционально)

add
        kubernetes-volumes/my-pod.yaml
        kubernetes-volumes/my-pv.yaml
        kubernetes-volumes/my-pvc.yaml

На worker нодах
sudo sh -c "echo 'Hello from Kubernetes storage' > /data/index.html"

sudo kubectl apply -f my-pv.yaml
persistentvolume/my-pv created

sudo kubectl apply -f my-pvc.yaml
persistentvolumeclaim/my-pvc created

sudo kubectl apply -f my-pod.yaml
pod/my-pod created

sudo kubectl get pvc
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
my-pvc   Bound    my-pv    1Gi        RWO            local-storage   14s

 sudo kubectl get pod
my-pod                     1/1     Running   0          9s

sudo kubectl exec -it my-pod -- /bin/bash
root@my-pod:/# curl http://localhost
Hello from Kubernetes storage

cd /usr/share/nginx/html
echo "snark" > data.txt

sudo kubectl apply -f my-pod2.yaml
pod/my-pod2 created

sudo kubectl get pod
my-pod2                    1/1     Running   0          27s

sudo kubectl exec -it my-pod2 -- /bin/bash

root@my-pod2:/# curl http://localhost
Hello from Kubernetes storage

root@my-pod2:/# cat /usr/share/nginx/html/data.txt
snark
