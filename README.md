# Домашнее задание к занятию «`Сетевое взаимодействие в K8S. Часть 1`» - `Живарев Игорь`

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

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---


## Ответ


### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

Запуск `Deployment` и `Service`  
![start](img/k8s-04_01.png)

Проверяем запуск `Service` и поднятие `Pod`  
![analyse](img/k8s-04_02.png)

Доступ с помощью `curl` по доменному имени сервиса  
![pod](img/k8s-04_03.png)

Листинг `Deployment`:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.2
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
          - name: HTTP_PORT
            value: "8080"
        ports:
        - containerPort: 8080
          name: http
```

Листинг `Service`:
```
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx-multitool
#  namespace: netology
spec:
  ports:
    - name: http-nginx
      port: 9001
      protocol: TCP
      targetPort: 80
    - name: http-multitool
      port: 9002
      protocol: TCP
      targetPort: 8080
  selector:
    app: web
```


### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

Новый `Service` и его запуск  
![service](img/k8s-04_05.png)

Доступ с помощью браузера на порт 30000  
![service](img/k8s-04_06.png)

Доступ с помощью браузера на порт 30001  
![service](img/k8s-04_07.png)


Листинг `Service`:
```
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx-multitool-2
#  namespace: netology
spec:
  ports:
    - name: http-nginx
      port: 80
      nodePort: 30000
    - name: http-multitool
      port: 8080
      nodePort: 30001
  selector:
    app: web
  type: NodePort
```

---
