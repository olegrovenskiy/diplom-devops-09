# Подготовка cистемы мониторинга и деплой приложения

Ожидаемый результат:

## 1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.

https://github.com/olegrovenskiy/diplom-k8s-config/tree/master

запуск плейбука

 ansible-playbook -i inventory/yandexcloud/inventory.ini cluster.yml --become --become-user=root


## 2. Http доступ к web интерфейсу grafana.

Воспользовался пакетом kube-prometheus, который уже включает в себя Kubernetes оператор для grafana, prometheus, alertmanager и node_exporter.
Использовал официальную документацию https://prometheus-operator.dev/docs/prologue/quick-start/

    root@mck-diplom-k8s-cp1:~# kubectl get services -n monitoring
    NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
    alertmanager-main       ClusterIP   10.233.61.115   <none>        9093/TCP,8080/TCP            117m
    alertmanager-operated   ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   117m
    blackbox-exporter       ClusterIP   10.233.6.176    <none>        9115/TCP,19115/TCP           117m
    grafana                 ClusterIP   10.233.2.0      <none>        3000/TCP                     117m
    kube-state-metrics      ClusterIP   None            <none>        8443/TCP,9443/TCP            117m
    node-exporter           ClusterIP   None            <none>        9100/TCP                     117m
    prometheus-adapter      ClusterIP   10.233.21.231   <none>        443/TCP                      117m
    prometheus-k8s          ClusterIP   10.233.31.87    <none>        9090/TCP,8080/TCP            117m
    prometheus-operated     ClusterIP   None            <none>        9090/TCP                     117m
    prometheus-operator     ClusterIP   None            <none>        8443/TCP                     117m
    root@mck-diplom-k8s-cp1:~#

http://51.250.3.203:3000/?orgId=1
admin / 0ber0n@2022

## 3. Дашборды в grafana отображающие состояние Kubernetes кластера.

http://51.250.3.203:3000/d/efa86fd1d0c121a26444b636a3f509a8/kubernetes-compute-resources-cluster?orgId=1&refresh=10s

![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/grafana.png)

## 4. Http доступ к тестовому приложению.

Для деплоя тестового приложения из образа в яндекс регистри сначала делаем секреты.

После логина проверяем файл:

    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# cat ~/.docker/config.json
    {
            "auths": {
                    "cr.yandex": {
                            "auth": "b2F1dGg6QVFBQUFBQmFwZWdFQUFUdXdRaU5lakw0ekVzY2hqdHJDamxXbHFV"
                    }
            }
    }root@mck-diplom-k8s-cp1:~/diplom/test-nginx#
  
 

Создаём кредентиалс

    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl create secret generic regcred \
    >                 --from-file=.dockerconfigjson=/root/.docker/config.json \
    >                 --type=kubernetes.io/dockerconfigjson
    secret/regcred created

Делаем тестовый манифест:


    apiVersion: v1
    kind: Pod
    metadata:
     name: test-nginx-diplom
    spec:
     containers:
     - name: test-nginx-diplom
       image: cr.yandex/crpafmldih9te92441ft/nginx:diplom
     imagePullSecrets:
     - name: regcred

Запускаем и проверяем под

    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl apply -f ./test-nginx-depl.yaml
    deployment.apps/test-nginx-diplom created
    service/test-nginx-diplom created
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx#
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx#
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx#
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx#
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get pods
    NAME                                 READY   STATUS    RESTARTS   AGE
    test-nginx-diplom-578479f76f-fmj8m   1/1     Running   0          18s
    test-nginx-diplom-578479f76f-h42dg   1/1     Running   0          18s
    test-nginx-diplom-578479f76f-ttkdt   1/1     Running   0          18s
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get pods -o wide
    NAME                                 READY   STATUS    RESTARTS   AGE   IP             NODE                 NOMINATED NODE   READINESS GATES
    test-nginx-diplom-578479f76f-fmj8m   1/1     Running   0          31s   10.233.123.4   mck-diplom-k8s-wn2   <none>           <none>
    test-nginx-diplom-578479f76f-h42dg   1/1     Running   0          31s   10.233.124.3   mck-diplom-k8s-wn1   <none>           <none>
    test-nginx-diplom-578479f76f-ttkdt   1/1     Running   0          31s   10.233.123.5   mck-diplom-k8s-wn2   <none>           <none>
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# curl localhost:8088
    curl: (7) Failed to connect to localhost port 8088: Connection refused
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get services
    NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    kubernetes          ClusterIP   10.233.0.1     <none>        443/TCP          36h
    test-nginx-diplom   NodePort    10.233.13.11   <none>        8088:32756/TCP   69s


Под  запустился, образ успешно скачен, значить можно готовить деплоймент

 
  Делаю деплоймент
  
     root@mck-diplom-k8s-cp1:~/diplom/test-nginx# cat test-nginx-depl.yaml
             apiVersion: apps/v1
             kind: Deployment
             metadata:
               labels:
                 app: test-nginx-diplom
               name: test-nginx-diplom
               namespace: default
             spec:
               replicas: 3
               selector:
                 matchLabels:
                   app: test-nginx-diplom
               template:
                  metadata:
                    labels:
                      app: test-nginx-diplom
                  spec:
                    containers:
                      - image: cr.yandex/crpafmldih9te92441ft/nginx:diplom
                        name: test-nginx-diplom
                    imagePullSecrets:
                      - name: regcred
             ---

             apiVersion: v1
             kind: Service
             metadata:
               name: test-nginx-diplom
               namespace: default
             spec:
               ports:
                 - name: web
                   port: 8088
                   targetPort: 80
                   nodePort: 32756
               selector:
                 app: test-nginx-diplom
               type: NodePort
   root@mck-diplom-k8s-cp1:~/diplom/test-nginx#


  Запускаю и проверяю  
  
  
      root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl apply -f ./test-nginx-depl.yaml
    deployment.apps/test-nginx-diplom created
    service/test-nginx-diplom created
  
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get pods
  NAME                                 READY   STATUS    RESTARTS   AGE
  test-nginx-diplom-578479f76f-fmj8m   1/1     Running   0          18s
  test-nginx-diplom-578479f76f-h42dg   1/1     Running   0          18s
  test-nginx-diplom-578479f76f-ttkdt   1/1     Running   0          18s
  root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get pods -o wide
  NAME                                 READY   STATUS    RESTARTS   AGE   IP             NODE                 NOMINATED NODE   READINESS GATES
  test-nginx-diplom-578479f76f-fmj8m   1/1     Running   0          31s   10.233.123.4   mck-diplom-k8s-wn2   <none>           <none>
  test-nginx-diplom-578479f76f-h42dg   1/1     Running   0          31s   10.233.124.3   mck-diplom-k8s-wn1   <none>           <none>
  test-nginx-diplom-578479f76f-ttkdt   1/1     Running   0          31s   10.233.123.5   mck-diplom-k8s-wn2   <none>           <none>

  
 Тесты курлом
 
    root@mck-diplom-k8s-cp1:~/diplom/test-nginx# curl 10.233.13.11:8088/images/image.webp
   Warning: Binary output can mess up your terminal. Use "--output -" to tell
   Warning: curl to output it to your terminal anyway, or consider "--output
   Warning: <FILE>" to save to a file.
   root@mck-diplom-k8s-cp1:~/diplom/test-nginx#
  
Всё успешно
 
Http ссылка к тестовому приложению
 
http://51.250.3.203:32756/images/image.webp
 
 



