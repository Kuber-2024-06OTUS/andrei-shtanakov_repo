# Проверка корректности применения манифестов

Для того, чтобы добиться цели, создал системный аккаунт для мониторинга, написал манифест для сервиса мониторинга, установил prometheus, создал манифест configmap.yaml, неоднократно переписывал deployment.yaml, nginx-config.yaml, и т.д.

Но главное было проверит, что работают метрики, собираются и их можно получить извне, и не сломался nginx.


## Проверка StorageClass
curl -k https://homework.otus/metrics
curl -k https://homework.otus/index.html
```
user@test:~/otus/kubernetes-security$ curl -k https://homework.otus/metrics
Active connections: 1
server accepts handled requests
 2 2 2
Reading: 0 Writing: 1 Waiting: 0
user@test:~/otus/kubernetes-security$ curl -k https://homework.otus/index.html
<html><body><h1>Hello, World!</h1></body></html>
user@test:~/otus/kubernetes-security$ curl -k https://homework.otus/metrics
Active connections: 1
server accepts handled requests
 19 19 19
Reading: 0 Writing: 1 Waiting: 0

```