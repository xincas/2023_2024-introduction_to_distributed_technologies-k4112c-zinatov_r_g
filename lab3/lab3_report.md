University: [ITMO University](https://itmo.ru/ru/) \
Faculty: [FICT](https://fict.itmo.ru) \
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies) \
Year: 2023/2024 \
Group: K4112с \
Author: Zinatov Rakhim Gamirovich \
Lab: Lab3 \
Date of create: 15.11.2023 \
Date of finished: <none>

### Запуск Minikube:
```
$ minikube minikube start
```

### Включение Ingress:
```
$ minikube addons enable ingress
```

Команда minikube addons enable ingress активирует дополнение (addon) под названием Ingress в Minikube. Ingress - это механизм в Kubernetes, который управляет внешним доступом к службам внутри кластера. Активация Ingress позволяет использовать эту функциональность для маршрутизации внешнего трафика в приложения в вашем локальном кластере Minikube.

### Создание сертификата:
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=xincas.me"
```

После выполнения этой команды создадутся 2 файла `tls.crt` и `tls.key`

## Создаем yaml файл:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: react-env
data:
  react_app_username: "xincas"
  react_app_company_name: "FICT ITMO"

---

apiVersion: apps/v1
kind: ReplicaSet                                            
metadata:
  name: react-repset                   
spec:
  replicas: 2
  selector:
    matchLabels:
      app: react
  template:
    metadata:
      labels:
        app: react
    spec:                                      
      containers:
        - image: ifilyaninitmo/itdt-contained-frontend:master
          name: react                           
          ports:
          - name: react-port
            containerPort: 3000
          env:
            - name: REACT_APP_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: react-env
                  key: react_app_username
            - name: REACT_APP_COMPANY_NAME
              valueFrom:
                configMapKeyRef:
                  name: react-env
                  key: react_app_company_name
        
---

apiVersion: v1
kind: Service
metadata:
  name: react-service
spec:
  selector:
    app: react
  type: ClusterIP
  ports:
    - port: 666
      name: react-port
      targetPort: react-port
      protocol: TCP

---

apiVersion: v1
kind: Secret
metadata:
  name: react-tls
type: kubernetes.io/tls
stringData:
  tls.crt: |
    -----BEGIN CERTIFICATE-----

    -----END CERTIFICATE-----
  tls.key: |
    -----BEGIN PRIVATE KEY-----

    -----END PRIVATE KEY----- 

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-ingress
spec:
  tls:
    - secretName: react-tls
      hosts:
        - xincas.me
  rules:
    - host: xincas.me
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: react-service
                port:
                  number: 666
```

Этот YAML файл описывает различные объекты Kubernetes, которые могут быть использованы для развертывания приложения.

## Добавим в файл hosts строку
```
127.0.0.1 xincas.me
```

## Применим файл конфигурации:
```
$ minikube kubectl apply -f d.yaml
```

Разворатывания приложения в локальном кластере Kubernetes, управляемом Minikube.

## Запустим tunnel 
```
$ minikube tunnel
```

Команда minikube tunnel используется для создания сетевого туннеля между вашим локальным кластером Minikube и сетью вашей машины.

## Открываем сайт:
![1](images/image1.png)

## Сертификат:
![2](images/image2.png)

## Схема:
![3](images/image3.drawio.png)