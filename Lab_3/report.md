University: [ITMO](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2024/2025  
Group: K4111c  
Author: [Russkin Vadim Denisovich](https://github.com/SolPot)  
Lab: [Laboratory Work №3 "Certificates and Secrets in Minikube, Secure Data Storage"](https://itmo-ict-faculty.github.io/introduction-to-distributed-technologies/education/labs2023_2024/lab3/lab3/)  
Date of create: 18.11.2024  
Date of finished: 19.11.2024  
### Ход работы  
1. Формирование [манифеста](configmap.yaml) для сервиса `ConfigMap`:  
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-configmap
data:
  react_app_user_name: "Russkin_Vadim"
  react_app_company_name: "ITMO"
```
2.Запуск миникуба с созданием configmap:
![](screenshot/1.png)
3.Проверка создания configmap:
![](screenshot/2.png)
4.Формирование [манифеста](replicaset.yaml) для контроллера `ReplicaSet`:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-replicaset
  labels:
    app: front
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front
  template:
    metadata:
      labels:
        app: front
    spec:
      containers:
      - name: frontend-container
        image: ifilyaninitmo/itdt-contained-frontend:master
        ports:
        - containerPort: 3000
        env:
        - name: REACT_APP_USERNAME
          valueFrom:
            configMapKeyRef:
              name: frontend-configmap
              key: react_app_user_name
        - name: REACT_APP_COMPANY_NAME
          valueFrom:
            configMapKeyRef:
              name: frontend-configmap
              key: react_app_company_name
```
5.Создание сервиса 'replicaset' и его проверка:
![](screenshot/3.png)
6.Формирование [манифеста](ingress.yaml) для сервиса `ingress`, и его создания:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: front
  labels:
    app: front
spec:
  type: NodePort
  selector:
    app: front
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 31111
'''
![](screenshot/4.png)
7.Установка [openssl]((https://slproweb.com/products/Win32OpenSSL.html)) и его [настройка](https://dev.to/danilovieira/installing-openssl-on-windows-and-adding-to-path-3mbf):
![](screenshot/5.png)
8. Генерация ключа и подпись сертификата на 30 дней:
![](screenshot/6.png)
Подпись
![](screenshot/7.png)
9.Создание tls
![](screenshot/8.png)
10.Включение аддонов ingress и ingress-dns:
![](screenshot/9.png)
11. Формирование манифеста 'front-ingress'
'''yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
spec:
  tls:
  - hosts:
      - fronted.edu
    secretName: fronted-tls
  rules:
  - host: fronted.edu
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 3000
'''
12. Проброс хоста в файл 'hosts'
13. Туннелируем Ingress
![](screenshot/10.png)
14.Переходим по адресу 'fronted.edu'
![](screenshot/11.png)
15.Проверка сведений о сертификате (спасибо касперскому с яндексу):
![](screenshot/12.png)
Схема организация контейнеров и сервисов
![](screenshot/13.png)
