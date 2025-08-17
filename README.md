# Домашнее задание к занятию " `Запуск приложений в K8S` " - `Сулименков Алексей`

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

---

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

---

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

---

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

---

### Ответ

1. Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool-deployment
  namespace: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
        - name: network-multitool
          image: wbitt/network-multitool:openshift
          ports:
            - containerPort: 1180
            - containerPort: 11443
      restartPolicy: Always
```

<details> <summary>Deployment</summary>

![run](https://github.com/biparasite/kuber-homeworks-01/blob/main/task_1.1.png "run")

</details>

2. scale deployment

```bash
kubectl scale deployment nginx-multitool-deployment --replicas=2 -n test
```

3.  Scale deployment

<details> <summary>scale deployment</summary>

![scale](https://github.com/biparasite/kuber-homeworks-01/blob/main/task_1.2.png "scale")

</details>

4. Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx-multitool
  namespace: test
spec:
  selector:
    app: nginx-multitool
  ports:
    - name: http-port-nginx
      protocol: TCP
      port: 80
      targetPort: 80
    - name: http-port-multitool
      protocol: TCP
      port: 1180
      targetPort: 1180
    - name: https-port-multitool
      protocol: TCP
      port: 11443
      targetPort: 11443
```

<details> <summary>svc</summary>

![svc](https://github.com/biparasite/kuber-homeworks-01/blob/main/task_1.3.png "svc")

</details>

5. Pod and curl

```bash
kubectl run network-multitool --image=wbitt/network-multitool:openshift -n test
```

<details> <summary>curl</summary>

![curl](https://github.com/biparasite/kuber-homeworks-01/blob/main/task_1.4.png "curl")

</details>

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

---

### Ответ

1. Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-init-deployment
  namespace: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-init
  template:
    metadata:
      labels:
        app: nginx-init
    spec:
      initContainers:
        - name: wait-for-service
          image: busybox
          command: [
              "sh",
              "-c",
              '
              until nslookup svc-nginx-init.test.svc.cluster.local; do
              echo "Waiting for service to start...";
              sleep 2;
              done;
              echo "Service is ready!"
              ',
            ]
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

2. POd Init

<details> <summary>INIT</summary>

![init](https://github.com/biparasite/kuber-homeworks-01/blob/main/task_2.1.png "init")

</details>

3. Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx-init
  namespace: test
spec:
  selector:
    app: nginx-init
  ports:
    - name: http-port
      protocol: TCP
      port: 80
      targetPort: 80
```

<details> <summary>svc</summary>

![svc](https://github.com/biparasite/kuber-homeworks-01/blob/main/task_2.3.png "svc")

</details>

4. Run pod

```bash
kubectl logs nginx-init-deployment-57898454c7-4bvgk -c wait-for-service  -n test
```

<details> <summary>logs init pod</summary>

```bash
Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

** server can't find svc-nginx-init.test.svc.cluster.local: NXDOMAIN

Waiting for service to start...
Server:         10.96.0.10
Address:        10.96.0.10:53


Name:   svc-nginx-init.test.svc.cluster.local
Address: 10.101.107.3

Service is ready!
```

</details>

<details> <summary>describe pod</summary>

```bash
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  7m58s  default-scheduler  Successfully assigned test/nginx-init-deployment-57898454c7-4bvgk to k8s-worker-node2
  Normal  Pulling    7m58s  kubelet            Pulling image "busybox"
  Normal  Pulled     7m57s  kubelet            Successfully pulled image "busybox" in 1.07s (1.07s including waiting). Image size: 2223685 bytes.
  Normal  Created    7m57s  kubelet            Created container wait-for-service
  Normal  Started    7m57s  kubelet            Started container wait-for-service
  Normal  Pulling    4m44s  kubelet            Pulling image "nginx"
  Normal  Pulled     4m43s  kubelet            Successfully pulled image "nginx" in 1.161s (1.161s including waiting). Image size: 72324501 bytes.
  Normal  Created    4m43s  kubelet            Created container nginx
  Normal  Started    4m42s  kubelet            Started container nginx
```

</details>

<details> <summary>run nginx</summary>

![nginx](https://github.com/biparasite/kuber-homeworks-01/blob/main/task_2.4.png "nginx")

</details>

---
