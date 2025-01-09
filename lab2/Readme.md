# Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4112

Author: Denisevich Vycheslav Petrovich

Lab: Lab2

Date of create: 16.12.2024

Date of finished: 16.12.2024

**1. Развернуть minikube cluster**
```
$ minikube start
```
**2. Манифест файла для развертывания deployment-react**

```
apiVersion: apps/v1
kind: Deployment                                            
metadata:
  name: deployment-react                         
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
              value: 'denisevich'
            - name: REACT_APP_COMPANY_NAME
              value: 'itmo'
```

**replicas:** 
Количество экземпляров приложения, которые должны быть запущены одновременно

**selector:** 
Определяет, какие поды будут управляться этим деплойментом

**3. Манифест файла для Service**
```
apiVersion: v1
kind: Service
metadata:
  name: front-service
spec:
  selector:
    app: react
  type: NodePort
  ports:
    - port: 3010
      name: react-port
      targetPort: 3000  # Используем порт, на котором работает приложение внутри контейнера
      protocol: TCP
```
**4. Привязывание манифеста**
```
$ kubectl apply -f service.yaml
```
**5. Перенаправление запрос с Service на pods**
```
$ minikube kubectl -- port-forward service/frontend-service 3010:3010
```

**6. Написать в браузере url**
```
http://localhost:3010
```
**7. Создать новый терминал и найти логи**
```
$ minikube kubectl -- get pods
NAME                                  READY   STATUS    RESTARTS        AGE
deployment-react-76cfb6b8b6-g4jjs     1/1     Running   0               7m
deployment-react-76cfb6b8b6-zrzcn     1/1     Running   0               7m
frontend-deployment-58cbffc88-8rrh6   1/1     Running   2 (7m50s ago)   70m
frontend-deployment-58cbffc88-wcww4   1/1     Running   2 (7m50s ago)   70m
vault                                 1/1     Running   3 (7m50s ago)   5d5h


$ minikube kubectl -- logs deployment-react-76cfb6b8b6-g4jjs
Builing frontend
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
build finished
Server started on port 3000

$ minikube kubectl -- logs deployment-react-76cfb6b8b6-zrzcn
Builing frontend
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
build finished
Server started on port 3000
```
![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab2/images/result_lab_2.jpg)

![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab2/images/lab2.drawio.jpg)

