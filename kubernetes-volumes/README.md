# Проверка корректности применения манифестов

Чтобы проверить, что все манифесты применены правильно, используйте следующие команды `kubectl`:

## Проверка StorageClass
kubectl get storageclass
kubectl describe storageclass homework-storage
```
user@test:~/otus/kubernetes-volumes$ kubectl get storageclass
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
homework-storage     k8s.io/minikube-hostpath   Retain          Immediate              false                  3m48s
standard (default)   rancher.io/local-path      Delete          WaitForFirstConsumer   false                  2d6h
user@test:~/otus/kubernetes-volumes$
user@test:~/otus/kubernetes-volumes$ kubectl describe storageclass homework-storage
Name:            homework-storage
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"homework-storage"},"provisioner":"k8s.io/minikube-hostpath","reclaimPolicy":"Retain"}

Provisioner:           k8s.io/minikube-hostpath
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Retain
VolumeBindingMode:     Immediate
Events:                <none>

```

## Проверка Namespace
kubectl get namespace homework

```
user@test:~/otus/kubernetes-volumes$ kubectl get namespace homework
NAME       STATUS   AGE
homework   Active   24m```
## Проверка PersistentVolumeClaim
kubectl get pvc -n homework
kubectl describe pvc homework-pvc -n homework

user@test:~/otus/kubernetes-volumes$ kubectl get pvc -n homework

NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
homework-pvc   Bound    pvc-f8c5bba8-1ee9-4a31-91a0-0ae08ba828d8   1Gi        RWO            standard       22m
user@test:~/otus/kubernetes-volumes$ kubectl describe pvc homework-pvc -n homework
Name:          homework-pvc
Namespace:     homework
StorageClass:  standard
Status:        Bound
Volume:        pvc-f8c5bba8-1ee9-4a31-91a0-0ae08ba828d8
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: rancher.io/local-path
               volume.kubernetes.io/selected-node: kind-control-plane
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       homework-deployment-5f9c6cbf7b-6k9qf
               homework-deployment-5f9c6cbf7b-m8dqc
               homework-deployment-5f9c6cbf7b-rcmsh
Events:
  Type    Reason                 Age                From
                  Message
  ----    ------                 ----               ----
                  -------
  Normal  WaitForFirstConsumer   22m (x4 over 22m)  persistentvolume-controller
                  waiting for first consumer to be created before binding
  Normal  ExternalProvisioning   22m                persistentvolume-controller
                  waiting for a volume to be created, either by external provisioner "rancher.io/local-path" or manually created by system administrator
  Normal  Provisioning           22m                rancher.io/local-path_local-path-provisioner-547f784dff-rw7zx_9b187a59-81d8-4e2d-a073-bb3043613f80  External provisioner is provisioning volume for claim "homework/homework-pvc"
  Normal  ProvisioningSucceeded  22m                rancher.io/local-path_local-path-provisioner-547f784dff-rw7zx_9b187a59-81d8-4e2d-a073-bb3043613f80  Successfully provisioned volume pvc-f8c5bba8-1ee9-4a31-91a0-0ae08ba828d8
```
## Проверка ConfigMap
kubectl get configmap -n homework
kubectl describe configmap homework-configmap -n homework
kubectl describe configmap nginx-config -n homework
```
user@test:~/otus/kubernetes-volumes$ kubectl get configmap -n homework
NAME                 DATA   AGE
homework-configmap   3      23m
kube-root-ca.crt     1      25m
nginx-config         1      23m
user@test:~/otus/kubernetes-volumes$ kubectl describe configmap homework-configmap -n homework
Name:         homework-configmap
Namespace:    homework
Labels:       <none>
Annotations:  <none>

Data
====
file:
----
This is a sample file content
that will be accessible via /conf/file

key1:
----
value1
key2:
----
value2

BinaryData
====

Events:  <none>
user@test:~/otus/kubernetes-volumes$ kubectl describe configmap nginx-config -n homework
Name:         nginx-config
Namespace:    homework
Labels:       <none>
Annotations:  <none>

Data
====
default.conf:
----
server {
    listen 80;
    server_name localhost;

    location / {
        root /homework;
        index index.html;
    }
}


BinaryData
====

Events:  <none>
```
## Проверка Deployment
kubectl get deployment -n homework
kubectl describe deployment homework-deployment -n homework

```
user@test:~/otus/kubernetes-volumes$ kubectl get deployment -n homework
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
homework-deployment   3/3     3            3           23m
user@test:~/otus/kubernetes-volumes$ kubectl describe deployment homework-deployment -n homework
Name:                   homework-deployment
Namespace:              homework
CreationTimestamp:      Sat, 20 Jul 2024 18:14:37 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=homework-pod
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=homework-pod
  Init Containers:
   init-container:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo "<html><body><h1>Hello, World!</h1></body></html>" > /homework/index.html
    Environment:  <none>
    Mounts:
      /homework from shared-volume (rw)
  Containers:
   web-server:
    Image:        nginx
    Port:         80/TCP
    Host Port:    0/TCP
    Readiness:    exec [test -f /homework/index.html] delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/nginx/conf.d from nginx-config (rw)
      /homework from shared-volume (rw)
      /homework/conf from homework-config (rw)
  Volumes:
   shared-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  homework-pvc
    ReadOnly:   false
   nginx-config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      nginx-config
    Optional:  false
   homework-config:
    Type:          ConfigMap (a volume populated by a ConfigMap)
    Name:          homework-configmap
    Optional:      false
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   homework-deployment-5f9c6cbf7b (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  24m   deployment-controller  Scaled up replica set homework-deployment-5f9c6cbf7b to 3
```
## Проверка Pods
kubectl get pods -n homework
```
user@test:~/otus/kubernetes-volumes$ kubectl get pods -n homework
NAME                                   READY   STATUS    RESTARTS   AGE
homework-deployment-5f9c6cbf7b-6k9qf   1/1     Running   0          24m
homework-deployment-5f9c6cbf7b-m8dqc   1/1     Running   0          24m
homework-deployment-5f9c6cbf7b-rcmsh   1/1     Running   0          24m
```
## Проверка Service
kubectl get service -n homework
kubectl describe service homework-service -n homework
```
user@test:~/otus/kubernetes-volumes$ kubectl get service -n homework
NAME               TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
homework-service   NodePort   10.96.86.9   <none>        80:31587/TCP   24m
user@test:~/otus/kubernetes-volumes$ kubectl describe service homework-service -n homework
Name:                     homework-service
Namespace:                homework
Labels:                   <none>
Annotations:              <none>
Selector:                 app=homework-pod
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.86.9
IPs:                      10.96.86.9
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31587/TCP
Endpoints:                10.244.0.11:80,10.244.0.12:80,10.244.0.13:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
## Проверка ValidatingWebhookConfiguration
kubectl get validatingwebhookconfiguration
kubectl describe validatingwebhookconfiguration ingress-nginx-admission
```
user@test:~/otus/kubernetes-volumes$ kubectl get validatingwebhookconfiguration
NAME                                    WEBHOOKS   AGE
ingress-nginx-admission                 1          16m
nginx-ingress-ingress-nginx-admission   1          18m
user@test:~/otus/kubernetes-volumes$ kubectl describe validatingwebhookconfiguration ingress-nginx-admission
Name:         ingress-nginx-admission
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  admissionregistration.k8s.io/v1
Kind:         ValidatingWebhookConfiguration
Metadata:
  Creation Timestamp:  2024-07-20T18:23:36Z
  Generation:          2
  Resource Version:    330032
  UID:                 a41d4da0-b12b-4706-99b0-9c8d9822968e
Webhooks:
  Admission Review Versions:
    v1
  Client Config:
    Ca Bundle:  LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkakNDQVJ5Z0F3SUJBZ0lSQU1QK3B2VVhKNDBrblR5ODFSQVIwYlV3Q2dZSUtvWkl6ajBFQXdJd0R6RU4KTUFzR0ExVUVDaE1FYm1sc01UQWdGdzB5TkRBM01qQXhPREU0TkRkYUdBOHlNVEkwTURZeU5qRTRNVGcwTjFvdwpEekVOTUFzR0ExVUVDaE1FYm1sc01UQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJHM0F0THRGCnM3a25SNVo3VjF0OHEzQTFzUVRON0J5bU9PbUh5UjdoTDRvcXZtL2hZUGRoekpvNExMNVBEWGx3a3doc2pWcFEKeVpSenlma3ZpdmdacUZXalZ6QlZNQTRHQTFVZER3RUIvd1FFQXdJQ0JEQVRCZ05WSFNVRUREQUtCZ2dyQmdFRgpCUWNEQVRBUEJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJTK3hzdWdCVE5lM0p5ZEUrR1FPYjNYCmdNWVloakFLQmdncWhrak9QUVFEQWdOSUFEQkZBaUVBeUtLSFJqam5ERWEwK0VDM3RpcW9lWkVNUUd6VG40Y2YKN2dmVTcxeFVVYjhDSUVJbGFTQ3F0N3prK1E3Q1YvbHF6ZEtjTGJVNGRRYjkvTDEzbEtqcXVvQU8KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    Service:
      Name:        ingress-nginx-controller-admission
      Namespace:   ingress-nginx
      Path:        /networking/v1/ingresses
      Port:        443
  Failure Policy:  Fail
  Match Policy:    Equivalent
  Name:            validate.nginx.ingress.kubernetes.io
  Namespace Selector:
  Object Selector:
  Rules:
    API Groups:
      networking.k8s.io
    API Versions:
      v1
    Operations:
      CREATE
      UPDATE
    Resources:
      ingresses
    Scope:          *
  Side Effects:     None
  Timeout Seconds:  10
Events:             <none>
```

Для каждой команды `describe` вы увидите детальную информацию о ресурсе, включая его текущее состояние, события и другие важные детали.

Чтобы убедиться, что все работает корректно, обратите внимание на следующее:

- Для Pods: убедитесь, что они в состоянии "Running"
- Для PVC: проверьте, что статус "Bound"
- Для Ingress: убедитесь, что указан правильный backend service
- Для StorageClass: проверьте, что provisioner указан правильно

Если вы хотите получить общую картину всех ресурсов в namespace, вы можете использовать:
kubectl get all -n homework
```
user@test:~/otus/kubernetes-volumes$ kubectl get all -n homework
NAME                                       READY   STATUS    RESTARTS   AGE
pod/homework-deployment-5f9c6cbf7b-6k9qf   1/1     Running   0          25m
pod/homework-deployment-5f9c6cbf7b-m8dqc   1/1     Running   0          25m
pod/homework-deployment-5f9c6cbf7b-rcmsh   1/1     Running   0          25m

NAME                       TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
service/homework-service   NodePort   10.96.86.9   <none>        80:31587/TCP   25m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/homework-deployment   3/3     3            3           25m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/homework-deployment-5f9c6cbf7b   3         3         3       25m
user@test:~/otus/kubernetes-volumes$
```

