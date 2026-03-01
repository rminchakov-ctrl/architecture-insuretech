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

# Применить конфигурацию
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa-memory.yaml
