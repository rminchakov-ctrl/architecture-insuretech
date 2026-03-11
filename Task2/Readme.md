# Подготовка

## requirements
pip3 install -r requirements.txt

## minikube
minikube start
minikube addons enable metrics-server

## scaletestapp
- клонировать репо scaletestapp
- собрать приложение под архитектуру Minikube
- - GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -o app .
- исправить docker файл на минимальный
- настроить Docker CLI на использование Docker Daemon от Minikube
- - eval $(minikube docker-env)
- собрать Docker образ внутри среды Minikube
- - docker build -t scaletestapp:local .

## Применить конфигурацию
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa-memory.yaml

kubectl apply -f ./prometeus/01-prometheus.yaml
kubectl apply -f ./prometeus/02-prometheus-adapter.yaml
kubectl apply -f ./prometeus/03-hpa-rps.yaml

## Что с мониторингом?
kubectl get pods -n monitoring

### доступные метрики
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .

### следим за HPA
kubectl get hpa test-app-hpa-rps -w

# Тестирование

## Память

``` bash
source locust-env/bin/activate
locust -f locustfile.py --host=http://localhost:8080 --headless -u 300 -r 20 -t 5m
```
[locust](/logs/mem-locust-log.txt)

[Результат](/logs/mem-hpa-log.txt)

## RPC

kubectl delete hpa test-app-hpa -n default

``` bash
source locust-env/bin/activate
locust -f locustfile.py --host=http://localhost:8080 --headless -u 300 -r 20 -t 5m
```
[locust](/logs/rpc-locust-log.txt)
[Результат](/logs/rpc-hpa-log.txt)