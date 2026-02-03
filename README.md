# Домашнее задание к занятию «Helm»

------

### Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.


### Решение 1. Подготовить Helm-чарт для приложения

Переменные Helm — это values.yaml и values-*.yaml в корне my-app чарта

![SCR-1](https://github.com/vladrabbit/K8S-6/blob/main/SCR/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-02-03%20%D0%B2%2014.20.51.png)

Шаблоны deploymetns находятся в директории templates

![SCR-2](https://github.com/vladrabbit/K8S-6/blob/main/SCR/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-02-03%20%D0%B2%2014.21.58.png)

Изменения версий делаются в переменых

values-*.yaml

- values-v1.yaml
    ```yaml
    backend:
    image:
        repository: myrepo/backend
        tag: "1.0.0"
    replicas: 1

    frontend:
    image:
        repository: myrepo/frontend
        tag: "1.0.0"
    replicas: 1
    ```

- values-v2.yaml

    ```yaml
    backend:
    image:
        repository: myrepo/backend
        tag: "2.0.0"
    replicas: 1

    frontend:
    image:
        repository: myrepo/frontend
        tag: "2.0.0"
    replicas: 1

    ```

Шаблоны deploymets при этом не меняются

- frontend-deployment.yaml 

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: {{ .Release.Name }}-frontend
    spec:
    replicas: {{ .Values.frontend.replicas }}
    selector:
        matchLabels:
        app: frontend
    template:
        metadata:
        labels:
            app: frontend
        spec:
        containers:
            - name: frontend
            image: "{{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}"
    ```

- backend-deployment.yaml 

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: {{ .Release.Name }}-backend
    spec:
    replicas: {{ .Values.backend.replicas }}
    selector:
        matchLabels:
        app: backend
    template:
        metadata:
        labels:
            app: backend
        spec:
        containers:
            - name: backend
            image: "{{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}"
    ```
------
### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.


### Решение 2. Запустить две версии в разных неймспейсах

- Создаем namespace

```bash
kubectl create namespace app1
kubectl create namespace app2
```

- Проверяем командой `kubectl get namespaces`

![SCR-3](https://github.com/vladrabbit/K8S-6/blob/main/SCR/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-02-03%20%D0%B2%2014.46.56.png)

- Запускаем приложения в разных namespace

 - Namespace app1 — версия 1

    ```bash
        helm install my-app-v1 . -n app1 -f values-v1.yaml
    ```

 - Namespace app1 — версия 2

    ```bash
        helm install my-app-v2 . -n app1 -f values-v2.yaml
    ```
 - Namespace app2 — версия 3

    ```bash
    helm install my-app-v3 . -n app2 -f values-v2.yaml
    ```

- Проверяем

    ```bash
    kubectl get deployments -n app1
    kubectl get deployments -n app2

    ```

![SCR-5](https://github.com/vladrabbit/K8S-6/blob/main/SCR/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-02-03%20%D0%B2%2014.54.15.png)

![SCR-6](https://github.com/vladrabbit/K8S-6/blob/main/SCR/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-02-03%20%D0%B2%2014.55.13.png)
