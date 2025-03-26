# Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4112с

Author: Denisevich Vycheslav Petrovich

Lab: Lab4

Date of create: 15.12.2024

Date of finished: 

**1. Установка плагина и режима работы Multi-Node Clusters**
```
$ minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode-demo
```
**2. Проверка работы Calico**

```
kubectl get pods -l k8s-app=calico-node -A
kubectl get nodes
```
![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/1.jpg)


**3. Пометка нод** 

--overwrite=true - если уже помечали nods до этого и хотите rename

```
$ kubectl label nodes multinode-demo-m02 rack=1 --overwrite=true
$ kubectl label nodes multinode-demo rack=0 --overwrite=true
```
![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/2.jpg)


**4. Манифест для Calico**

```
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: rack-0
spec:
  cidr: 192.168.10.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "0"
---
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: rack-1
spec:
  cidr: 192.168.20.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "1"
```

**5. Скачивание конфигурационного файла**

**(перед командой необходимо [скачать конфигурационный файл)](https://github.com/projectcalico/calico/blob/master/manifests/calicoctl.yaml)**
```
$ curl.exe -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```
![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/3.jpg)

**6. Создание deployment**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
  labels:
    app: lab4
spec:
  replicas: 2
  selector: 
    matchLabels:
      app: lab4
  template:
    metadata:
      labels:
        app: lab4
    spec:
      containers:
      - name: lab4
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          value: denisevich
        - name: REACT_APP_COMPANY_NAME
          value: itmo
        ports:
        - containerPort:  3000

```

**7. Создание service**
```
apiVersion: v1
kind: Service
metadata:
  name: service-lab4
spec:
  type: NodePort
  selector:
    app: lab4
  ports:
    - port: 3000
      protocol: TCP
      name: http
```

**8. Обновление deployment и service**
```
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
```

**9. Вход в веб браузер**
```
localhost:3000
```

![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/4.jpg)

![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/5.jpg)


**10. Узнать информацию о FQDN имена подов**
```
$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP               NODE                 NOMINATED NODE   READINESS GATES
deployment-6dd4bd667f-8pdlf   1/1     Running   0          5m31s   10.244.239.2     multinode-demo-m02   <none>           <none>
deployment-6dd4bd667f-zzrnz   1/1     Running   0          5m31s   10.244.113.195   multinode-demo       <none>           <none>          <none>


$ kubectl exec deployment-6dd4bd667f-8pdlf -- nslookup 10.244.239.2
Server:         10.96.0.10
Address:        10.96.0.10:53

2.239.244.10.in-addr.arpa       name = 10-244-239-2.service-lab4.default.svc.cluster.local


$ kubectl exec deployment-6dd4bd667f-zzrnz  -- nslookup 10.244.113.195
Server:         10.96.0.10
Address:        10.96.0.10:53

195.113.244.10.in-addr.arpa     name = 10-244-113-195.service-lab4.default.svc.cluster.local
```

![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/6.jpg)


**11. Пинг подов**
```
$ kubectl exec -it deployment-6dd4bd667f-8pdlf -- sh
/frontend # ping 10-244-113-195.service-lab4.default.svc.cluster.local
PING 10-244-113-195.service-lab4.default.svc.cluster.local (10.244.113.195): 56 data bytes
64 bytes from 10.244.113.195: seq=0 ttl=62 time=0.308 ms
64 bytes from 10.244.113.195: seq=1 ttl=62 time=0.137 ms
64 bytes from 10.244.113.195: seq=2 ttl=62 time=0.133 ms
64 bytes from 10.244.113.195: seq=3 ttl=62 time=0.307 ms
64 bytes from 10.244.113.195: seq=4 ttl=62 time=0.253 ms
64 bytes from 10.244.113.195: seq=5 ttl=62 time=0.256 ms
64 bytes from 10.244.113.195: seq=6 ttl=62 time=0.245 ms
64 bytes from 10.244.113.195: seq=7 ttl=62 time=0.151 ms

$ kubectl exec -it deployment-6dd4bd667f-zzrnz -- sh
/frontend # ping 10-244-239-2.service-lab4.default.svc.cluster.local
PING 10-244-239-2.service-lab4.default.svc.cluster.local (10.244.239.2): 56 data bytes
64 bytes from 10.244.239.2: seq=0 ttl=62 time=0.158 ms
64 bytes from 10.244.239.2: seq=1 ttl=62 time=0.360 ms
64 bytes from 10.244.239.2: seq=2 ttl=62 time=0.177 ms
64 bytes from 10.244.239.2: seq=3 ttl=62 time=0.376 ms
64 bytes from 10.244.239.2: seq=4 ttl=62 time=0.179 ms
64 bytes from 10.244.239.2: seq=5 ttl=62 time=0.235 ms
64 bytes from 10.244.239.2: seq=6 ttl=62 time=0.187 ms
64 bytes from 10.244.239.2: seq=7 ttl=62 time=0.191 ms
64 bytes from 10.244.239.2: seq=8 ttl=62 time=0.280 ms
64 bytes from 10.244.239.2: seq=9 ttl=62 time=0.399 ms
```
![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/7.jpg)

![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/8.jpg)

![image](https://github.com/VACHESLAVE/VACHESLAVE-VACHESLAVE-2023_2024-introduction_to_distributed_technologies-k4112c-denisevich_v_p/blob/main/lab4/image/9.jpg)
