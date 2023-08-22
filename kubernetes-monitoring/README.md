HW Мониторинг сервиса в кластере k8s

kubectl create ns web

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo add grafana https://grafana.github.io/helm-charts

helm repo update

kubectl create ns monitoring

helm install prometheus prometheus-community/prometheus -n monitoring

kubectl get all -n monitoring

 helm install grafana grafana/grafana -n monitoring

kubectl apply -f deployment.yaml
kubectl apply -f configmap.yaml
kubectl apply -f service.yaml

kubectl get all -n web

kubectl apply -f deployment-exporter.yaml
kubectl apply -f svc-exporter.yaml

kubectl get all -n web

 export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace monitoring port-forward $POD_NAME 9090

export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")
     kubectl --namespace monitoring port-forward $POD_NAME 3000

 kubectl --namespace monitoring port-forward grafana-75977d9786-wz56p 3000 --address 0.0.0.0

 kubectl --namespace monitoring port-forward $POD_NAME 9090 --address 0.0.0.0

kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

add panel from nginx-exporter to dashboard

