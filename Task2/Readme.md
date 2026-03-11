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

## Невысокая нагрузка
Если locust -f locustfile.py --host=http://localhost:8080 --headless -u 100 -r 100 -t 10m
Нагрузка на памят не растет - 42%

## Высокая нагрузка
Если locust -f locustfile.py --host=http://localhost:8080 --headless -u 1000 -r 100 -t 10m
Виртуалка виснет - sudo reboot :(
После этого - pod-desc.txt
Ну и после - переустановка машины - vm.md