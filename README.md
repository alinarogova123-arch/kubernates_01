# Django Site

## Запуск локально через minikube

Вам понадобится установленный Docker, minikube, kubectl.

Запустите кластер minikube, используя нужную виртуальную машину:

```
minikube start
```

Включите ingress контроллер:

```
minikube addons enable ingress
```

Загрузите собранный вами образ вашего django-проекта в кластер:

```
minikube image load "Имя_образа"
```

Если установить базу данных с помощью helm не удается, cкачайте образ postgres 15 локально и загрузите в кластер minikube образ базы данных:

```
minikube image load postgres:15
```

Перейдите в директорию `kubernetes`

Создайте файл `django-secret.yaml` и задайте в нем следующие значения, используя секцию `stringData`:

- secret-key: секретный ключ сайта.
- database-url: "postgres://имя_пользователя:пароль@postgres-service:5432/mydb"
- django-debug: режим отладки.
- django-allowed-hosts: домены и IP-адреса сайта.
- postgres-user: имя пользователя базы данных.
- postgres-password: пароль пользователя базы данных.

Задайте имя вашего образа с проектом в поле `image` в следующих файлах:
- `django-deployment.yaml`
- `django-clearsessions-cronjob.yaml`
- `django-migrate-job.yaml`

Создайте секреты кластера:

```
kubectl apply -f django-secret.yaml
```

Запустите базу данных:

```
kubectl apply -f postgres-k8s.yaml
```

Запустите миграции внутри базы данных:

```
kubectl apply -f django-migrate-job.yaml
```

Запустите django и его сервис:

```
kubectl apply -f django-deployment.yaml
```
```
kubectl apply -f django-service.yaml
```

Запустите планировщик очистки сессий:

```
kubectl apply -f django-clearsessions-cronjob.yaml
```

Включите маршрутизацию для внешних запросов:

```
kubectl apply -f django-ingress.yaml
```

Включите сетевой туннель:

```
minikube tunnel
```

В файле C:\Windows\System32\drivers\etc\hosts в самом низу допишите следующую строку:

```
127.0.0.1 star-burger.test
```

Ваш сайт будет доступен по адресу [http://star-burger.test/]

## Как подготовить dev окружение.

### 1. Создайте secret файл с сертификатом для доступа к базе данных:

Используйте Lens, зайдите в config -> secrets, нажмите кнопку создания нового secrets, задайте значения в окнах SECRET NAME и NAMESPACE, нажмите create, отредактируйте файл - добавьте в него секцию `stringData`, задайте для ключа `root.crt` многострочный текст вашего сертификата:

```
type: Opaque
stringData:
  root.crt: |
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----
```

## Цели проекта

Код написан в учебных целях — для курса по Python и веб-разработке на сайте [Devman](https://dvmn.org).
