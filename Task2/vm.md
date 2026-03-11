ssh -i ~/.ssh/yandex_cloud_id_ed25519 rminchakov@158.160.208.248

sudo apt-get update
sudo apt-get upgrade -y
 
sudo apt-get install -y docker.io

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

sudo usermod -aG docker $USER
newgrp docker

minikube start --disk-size=20g --memory=2048mb --driver=docker
minikube addons enable metrics-server

# проверим, что метрики включены
minikube addons list | grep metrics-server
    *│ metrics-server              │ minikube │ enabled ✅ │ Kubernetes                             │*
kubectl get pods -n kube-system | grep metrics-server
    *metrics-server-9d74bb658-lkdvf     1/1     Running   0            37s*

# скопировал манифесты
scp -i ~/.ssh/yandex_cloud_id_ed25519 -r ./ rminchakov@158.160.208.248:/home/rminchakov/

kubectl apply -f deployment.yaml
    *deployment.apps/test-app created*
kubectl apply -f service.yaml
    *service/test-app-service created*
kubectl apply -f hpa-memory.yaml
    *horizontalpodautoscaler.autoscaling/test-app-hpa created*

# Установка venv (если не установлен)
sudo apt install python3-venv -y
# Создание виртуального окружения
python3 -m venv locust-env
# Активация виртуального окружения
source locust-env/bin/activate
## Теперь Locust внутри виртуального окружения
pip install locust

# port-forward приложения
kubectl port-forward svc/test-app-service 8080:80

# грузим
source locust-env/bin/activate
locust -f locustfile.py --host=http://localhost:8080 --headless -u 300 -r 20 -t 2m

## desc
kubectl describe hpa test-app-hpa -n default

## текущее состояние HPA
kubectl get hpa test-app-hpa -o wide
    *test-app-hpa   Deployment/test-app   memory: 85%/80%   1         10        1          12m*

## История scaling-событий
kubectl get events --field-selector involvedObject.name=test-app-hpa --sort-by=.lastTimestamp

## Логи metrics-server (Если HPA не масштабируется)
kubectl logs -n kube-system deployment/metrics-server

# prometeus
kubectl apply -f ./prometeus/01-prometheus.yaml
    *namespace/monitoring created*
    *deployment.apps/prometheus created*
    *service/prometheus created*
    *configmap/prometheus-config created*
kubectl apply -f ./prometeus/02-prometheus-adapter.yaml
    *deployment.apps/prometheus-adapter created*
    *service/prometheus-adapter created*
    *configmap/prometheus-adapter-config created*
    *rolebinding.rbac.authorization.k8s.io/prometheus-adapter-auth-reader created*
    *apiservice.apiregistration.k8s.io/v1beta1.custom.metrics.k8s.io created*
    *clusterrolebinding.rbac.authorization.k8s.io/prometheus-adapter-resource-reader created*
    *clusterrolebinding.rbac.authorization.k8s.io/prometheus-adapter-delegated-auth created*
kubectl apply -f ./prometeus/03-hpa-rps.yaml
    *horizontalpodautoscaler.autoscaling/test-app-hpa-rps created*

# доступные метрики
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .
``` bash
    {
        "kind": "APIResourceList",
        "apiVersion": "v1",
        "groupVersion": "custom.metrics.k8s.io/v1beta1",
        "resources": []
    }
```

# следим за HPA
kubectl get hpa test-app-hpa-rps -w