**1. Развернуть minikube cluster**
```
$ minikube start
```
**2. Манифест файла для развертывания vault**

```
apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
    app: vault
spec:
  containers:
  - name: vault
    image: hashicorp/vault:latest
    ports:
    - containerPort: 8200
```

**apiVersion:** v1
Указывает версию API Kubernetes

**kind:** Pod
Определяет тип создаваемого объекта.

**metadata**
Содержит метаинформацию о Pod

**name:** vault — имя  используется для идентификации

**labels:** app: vault — метки  для классификации Pod

**spec**
Описывает спецификацию Pod

**containers** — список контейнеров, которые будут запущены в Pod.

**image:** — образ контейнера  

**name:** имя контейнера.

**ports** открытые порты

**name:** vault-port — название порта

**containerPort:** 8200 — порт внутри контейнера, который будет доступен для работы 

**3. Создание или обновление объекта в Pod**
```
$ kubectl apply -f vault.yaml
```
**4. Создаёт объект Service, который открывает доступ к Pod vault через порт 8200**
```
$ minikube kubectl -- expose pod vault --type=NodePort --port=8200
```
**5. Перенаправление локального порта 8200 на порт 8200 сервиса vault, чтобы получить доступ к сервису**
```
$ minikube kubectl -- port-forward service/vault 8200:8200
```
**6. Написать в браузере url**
```
http://localhost:8200/ui/vault/dashboard
```
**7. Создать новый терминал и найти Root Token**
```
$ minikube kubectl logs vault
```

![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab1/images/result_lab_1.jpg)

![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab1/images/lab1.drawio.jpg)




