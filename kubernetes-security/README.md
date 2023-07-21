
6. Security

sudo kubectl apply -f 01-SA-bob.yaml
sudo kubectl get sa
NAME      SECRETS   AGE
bob       0         5s

sudo kubectl apply -f 02-rolebinding-bob.yaml
clusterrolebinding.rbac.authorization.k8s.io/bind-bob-admin created



sudo kubectl get ClusterRoleBinding | grep bob
bind-bob-admin                                         ClusterRole/admin                                                                  2m32s

sudo kubectl apply -f 03-SA-dave.yaml

sudo kubectl get sa
NAME      SECRETS   AGE
bob       0         7m37s
dave      0         2m48s


sudo kubectl describe  ClusterRoleBinding bind-bob-admin
Name:         bind-bob-admin
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  admin
Subjects:
  Kind            Name  Namespace
  ----            ----  ---------
  ServiceAccount  bob   default


task 02

sudo kubectl apply -f 01-namespace.yaml
namespace/prometheus created

sudo kubectl get ns | grep pr
prometheus        Active   43s


sudo kubectl apply -f 02-SA-carol.yaml
serviceaccount/carol created

 sudo kubectl get sa -n prometheus
NAME      SECRETS   AGE
carol     0         40s

sudo kubectl apply -f 03-role.yaml
clusterrole.rbac.authorization.k8s.io/prometheus-role created

sudo kubectl get clusterrole -n prometheus | grep prometheus-role
prometheus-role                                                        2023-07-21T21:57:11Z

sudo kubectl apply -f 04-rolebinding.yaml
clusterrolebinding.rbac.authorization.k8s.io/serviceaccount-prometheus created

sudo kubectl get clusterrolebinding | grep serviceaccount-prometheus
serviceaccount-prometheus                              ClusterRole/prometheus-role

task03

sudo kubectl apply -f 01-namespace.yaml

sudo kubectl get ns
NAME              STATUS   AGE
default           Active   5d1h
dev               Active   18s

 sudo kubectl apply -f 02-SA-jane.yaml
serviceaccount/jane created

sudo kubectl get sa -n dev
NAME      SECRETS   AGE
default   0         50s
jane      0         12s

sudo kubectl apply -f 03-role-admin.yaml
role.rbac.authorization.k8s.io/admin-ns-dev created

 sudo kubectl get role -n dev
NAME           CREATED AT
admin-ns-dev   2023-07-21T22:03:22Z

 sudo kubectl apply -f 04-rolebinding-jane.yaml
rolebinding.rbac.authorization.k8s.io/jane created

sudo kubectl get rolebinding -n dev
NAME   ROLE                AGE
jane   Role/admin-ns-dev   26s

sudo kubectl apply -f 05-SA-ken.yaml
serviceaccount/ken created

sudo kubectl get sa -n dev
NAME      SECRETS   AGE
default   0         3m30s
jane      0         2m52s
ken       0         21s

sudo kubectl apply -f 06-role-view.yaml
role.rbac.authorization.k8s.io/viewer-ns-dev created

sudo kubectl get role -n dev
NAME            CREATED AT
admin-ns-dev    2023-07-21T22:03:22Z
viewer-ns-dev   2023-07-21T22:05:48Z

sudo kubectl apply -f 07-rolebinding-ken.yaml
rolebinding.rbac.authorization.k8s.io/ken-viewer created

sudo kubectl get rolebinding -n dev
NAME         ROLE                 AGE
jane         Role/admin-ns-dev    2m19s
ken-viewer   Role/viewer-ns-dev   19s