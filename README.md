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



