# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера
Отредактировал манифест из прошлого задания mydeployment.yaml:
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: md
spec:
  replicas: 3
  selector:
    matchLabels:
      app: md
  template:
    metadata:
      labels:
        app: md
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
        - name: HTTP_PORT
          value: "8080"
```
Обновил и  и проверил:
```bash
user@microk8s:~/kuber-homeworks-1.4$ kubectl get deployments.apps 
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   3/3     3            3           3d
```
Отредактировал манифест md-svc.yaml:
```yml
apiVersion: v1
kind: Service
metadata:
  name: md-svc
spec:
  selector:
    app: md
  ports:
    - protocol: TCP
      name: nginx
      port: 9001
      targetPort: 80
    - protocol: TCP
      name: multitool
      port: 9002
      targetPort: 8080
```
Проверил service:
```bash
user@microk8s:~/kuber-homeworks-1.4$ kubectl get service
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP             8d
md-svc       ClusterIP   10.152.183.207   <none>        9001/TCP,9002/TCP   60m
```
Использовал манифест multitool.yaml из прошлого задания:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: multitool
spec:
  containers:
    - name: multitool
      image: wbitt/network-multitool
      ports:
        - containerPort: 8080
```
Зашел в pod и проверил доступность приложений по портам:
```bash
user@microk8s:~/kuber-homeworks-1.4$ kubectl exec multitool -ti -- bash
multitool:/# curl md-svc:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
multitool:/# curl md-svc:9002
WBITT Network MultiTool (with NGINX) - my-deployment-7dd9464df6-qv7nb - 10.1.128.243 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера
Создал манифест out-service.yaml:
```yml
apiVersion: v1
kind: Service
metadata:
  name: out-service
spec:
  selector:
    app: md
  ports:
    - protocol: TCP
      name: nginx
      port: 80
      nodePort: 30001
  type: NodePort
```
Применил и проверил:
```bash
user@microk8s:~/kuber-homeworks-1.4$ kubectl apply -f out-service.yaml 
service/out-service created
user@microk8s:~/kuber-homeworks-1.4$ kubectl get se
secrets          serviceaccounts  services         
user@microk8s:~/kuber-homeworks-1.4$ kubectl get service
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
kubernetes    ClusterIP   10.152.183.1     <none>        443/TCP             8d
md-svc        ClusterIP   10.152.183.207   <none>        9001/TCP,9002/TCP   77m
out-service   NodePort    10.152.183.182   <none>        80:30001/TCP        10s
```
Проверил доступ из вне:


------

