# Создание helm-chart позволяющего деплоить приложение, которое у вас получилось при выполнении предыдущих ДЗ 1-5.

Для создания helm-chart, создал папку templates, перенес туда все .yaml файлы, создал в текущем каталоге файл values.yaml с параметрами, параметризовал файлы в папке templates.

## Проверка работы приложения
kubectl describe service my-release-deployment -n homework

curl http://homework.otus

curl -k https://homework.otus/index.html

curl http://homework.otus/metrics

В процессе проверки отредактировал файл "/etc/hosts"

```
user@test:~/otus/kubernetes-templating$  kubectl describe service my-release-deployment -n homework
Name:              my-release-deployment
Namespace:         homework
Labels:            app.kubernetes.io/managed-by=Helm
Annotations:       <none>
Selector:          app=my-release
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.105.232.189
IPs:               10.105.232.189
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.10:80,10.244.1.11:80,10.244.1.12:80
Session Affinity:  None
Events:            <none>
user@test:~/otus/kubernetes-templating$ curl http://10.105.232.189
<html><body><h1>Hello, World!</h1></body></html>
user@test:~/otus/kubernetes-templating$ sudo vim /etc/hosts
[sudo] password for user:
user@test:~/otus/kubernetes-templating$ curl http://homework.otus
<html><body><h1>Hello, World!</h1></body></html>
user@test:~/otus/kubernetes-templating$ curl http://homework.otus/metrics
Active connections: 1
server accepts handled requests
 90 90 90
Reading: 0 Writing: 1 Waiting: 0
user@test:~/otus/kubernetes-templating$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 test
10.105.232.189 homework.otus

user@test:~/otus/kubernetes-templating$ minikube ip
192.168.58.2
user@test:~/otus/kubernetes-templating$
```

# Установить kafka из bitnami helm-чарта

Создал дополнительную папку helm_file и в ней создал helmfile.yaml

## Проверка работы 
```
user@test:~/otus/kubernetes-templating/helm_file$ helm get values kafka-dev -n dev
USER-SUPPLIED VALUES:
auth:
  clientProtocol: plaintext
  interBrokerProtocol: plaintext
broker:
  replicaCount: 0
controller:
  replicaCount: 1
persistence:
  enabled: true
  size: 1Gi
  storageClass: local-path
plaintext:
  enabled: true
sasl:
  enabled: false
user@test:~/otus/kubernetes-templating/helm_file$ helm get values kafka-prod -n prod
USER-SUPPLIED VALUES:
auth:
  clientProtocol: sasl
  interBrokerProtocol: sasl
broker:
  replicaCount: 0
controller:
  replicaCount: 5
image:
  tag: 3.6.0
livenessProbe:
  initialDelaySeconds: 120
persistence:
  enabled: true
  size: 1Gi
  storageClass: local-path
plaintext:
  enabled: false
readinessProbe:
  initialDelaySeconds: 120
sasl:
  controllerMechanism: plain
  enabledMechanisms: plain
  interBrokerMechanism: plain
user@test:~/otus/kubernetes-templating/helm_file$ kubectl get pods -n dev
NAME                     READY   STATUS    RESTARTS   AGE
kafka-dev-controller-0   1/1     Running   0          26m
user@test:~/otus/kubernetes-templating/helm_file$ kubectl get pods -n prod
NAME                      READY   STATUS             RESTARTS      AGE
kafka-prod-controller-0   1/1     Running            0             26m
kafka-prod-controller-1   1/1     Running            0             26m
kafka-prod-controller-2   1/1     Running            0             26m
kafka-prod-controller-3   1/1     Running            0             26m
kafka-prod-controller-4   1/1     Running            0             26m
user@test:~/otus/kubernetes-templating/helm_file$ 
```