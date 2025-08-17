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

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

---
