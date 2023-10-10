HW kubernetes-operators

kubectl apply -f deploy/cr.yml
error: unable to recognize "deploy/cr.yml": no matches for kind "MySQL" in version "otus.homework/v1"
kubectl apply -f deploy/crd.yml
The CustomResourceDefinition "mysqls.otus.homework" is invalid: spec.versions[0].schema.openAPIV3Schema: Required value: schemas are required

add schema
kubectl apply -f deploy/crd.yml
customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework created

kubectl apply -f deploy/cr.yml
Error from server (BadRequest): error when creating "deploy/cr.yml": MySQL in version "v1" cannot be handled as a MySQL: strict decoding error: unknown field "usless_data"
rm
kubectl apply -f deploy/cr.yml
mysql.otus.homework/mysql-instance created

kubectl get crd
NAME                           CREATED AT
addresspools.metallb.io        2023-07-17T15:28:51Z
bfdprofiles.metallb.io         2023-07-17T15:28:51Z
bgpadvertisements.metallb.io   2023-07-17T15:28:51Z
bgppeers.metallb.io            2023-07-17T15:28:51Z
communities.metallb.io         2023-07-17T15:28:51Z
ipaddresspools.metallb.io      2023-07-17T15:28:51Z
l2advertisements.metallb.io    2023-07-17T15:28:51Z
mysqls.otus.homework           2023-07-31T22:49:58Z

kubectl get mysqls.otus.homework
NAME             AGE
mysql-instance   57s

kubectl describe mysqls.otus.homework mysql-instance
Name:         mysql-instance
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  otus.homework/v1
Kind:         MySQL
Metadata:
  Creation Timestamp:  2023-07-31T22:52:01Z
  Generation:          1
  Resource Version:    427654
  UID:                 9922df5f-c181-4f6c-a009-0fa8219abd05
Spec:
  Database:      otus-database
  Image:         mysql:5.7
  Password:      otuspassword
  storage_size:  1Gi
Events:          <none>

kubectl delete mysqls.otus.homework mysql-instance
mysql.otus.homework "mysql-instance" deleted


add CRD ( spec ) параметры validation

kubectl apply -f deploy/crd.yml
kubectl apply -f deploy/cr.yml

kopf run mysql-operator.py
/home/sop/.local/lib/python3.10/site-packages/kopf/_core/reactor/running.py:179: FutureWarning: Absence of either namespaces or cluster-wide flag will become an error soon. For now, switching to the cluster-wide mode for backward compatibility.
  warnings.warn("Absence of either namespaces or cluster-wide flag will become an error soon."
[2023-07-31 23:17:41,268] kopf._core.engines.a [INFO    ] Initial authentication has been initiated.
[2023-07-31 23:17:41,277] kopf.activities.auth [INFO    ] Activity 'login_via_client' succeeded.
[2023-07-31 23:17:41,277] kopf._core.engines.a [INFO    ] Initial authentication has finished.
[2023-07-31 23:17:42,391] kopf.objects         [INFO    ] [default/mysql-instance] Handler 'mysql_on_create' succeeded.
[2023-07-31 23:17:42,392] kopf.objects         [INFO    ] [default/mysql-instance] Creation is processed: 1 succeeded; 0 failed.

Вопрос: почему объект создался, хотя мы создали CR, до того, как
запустили контроллер?

Причина в контроллере. Он сравнивает состояние ресурсов с желаемым состоянием из etcd
В случае разницы пытается исправить

kubectl delete mysqls.otus.homework mysql-instance
mysql.otus.homework "mysql-instance" deleted
kubectl delete deployments.apps mysql-instance
deployment.apps "mysql-instance" deleted
kubectl delete pvc mysql-instance-pvc
persistentvolumeclaim "mysql-instance-pvc" deleted
kubectl delete pv mysql-instance-pv
persistentvolume "mysql-instance-pv" deleted
kubectl delete svc mysql-instance
service "mysql-instance" deleted

kopf run mysql-operator.py
kubectl apply -f deploy/cr.yml

kubectl get pvc
NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-mysql-instance-pvc   Bound    pvc-1b7b0522-da8a-4dfd-9e39-5e0aea901a30   1Gi        RWO            standard       13s
mysql-instance-pvc          Bound    pvc-774ccd68-68d2-4357-b7c0-b84d6c0b7c0b   1Gi        RWO            standard       14s

export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath=" {.items[*].metadata.name}")


kubectl exec -it $MYSQLPOD -- mysql -u root -potuspassword -e "CREATE TABLE test (id smallint unsigned not null auto_increment, name varchar(20) not null, constraint pk_example primary key (id) );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name) VALUES ( null, 'some data' );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data-2' );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database

+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+


kubectl delete mysqls.otus.homework mysql-instance
mysql.otus.homework "mysql-instance" deleted

kubectl get jobs.batch
NAME                        COMPLETIONS   DURATION   AGE
backup-mysql-instance-job   1/1           49s        115s

kubectl apply -f deploy/cr.yml
mysql.otus.homework/mysql-instance created

export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")
kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database

+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+

add Dockerfile
docker build
docker taf
docker push

kubectl delete mysqls.otus.homework mysql-instance
kubectl delete deployments.apps mysql-instance
kubectl delete pvc mysql-instance-pvc
kubectl delete pv mysql-instance-pv
kubectl delete svc mysql-instance
export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")
 kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "DROP table test;" otus-database

add files service-account.yml role.yml role-binding.yml deploy-operator.yml

kubectl apply -f service-account.yml
serviceaccount/mysql-operator created

 kubectl apply -f role.yml
clusterrole.rbac.authorization.k8s.io/mysql-operator created

kubectl apply -f role-binding.yml
clusterrolebinding.rbac.authorization.k8s.io/workshop-operator created

kubectl apply -f deploy-operator.yml
deployment.apps/mysql-operator created

kubectl get pvc

kubectl get pvc
NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-mysql-instance-pvc   Bound    pvc-619fe046-86e9-411c-9106-38c4ae5bc268   1Gi        RWO            standard       5s
mysql-instance-pvc          Bound    pvc-ac116fb9-0982-4aed-afd6-7f338bd3c4c8   1Gi        RWO            standard       5s

export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")

kubectl exec -it $MYSQLPOD -- mysql -u root -potuspassword -e "CREATE TABLE test (id smallint unsigned not null auto_increment, name varchar(20) not null, constraint pk_example primary key (id) );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name) VALUES ( null, 'some data' );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data-2' );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database

kubectl delete mysqls.otus.homework mysql-instance

kubectl get pv

kubectl get jobs.batch
NAME                        COMPLETIONS   DURATION   AGE
backup-mysql-instance-job   1/1           19s        68s

kubectl apply -f deploy/cr.yml
export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")
kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
