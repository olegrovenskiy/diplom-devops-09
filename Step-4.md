# Подготовка cистемы мониторинга и деплой приложения

Ожидаемый результат:

## 1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.

https://github.com/olegrovenskiy/diplom-k8s-config/tree/master

запуск плейбука

 ansible-playbook -i inventory/yandexcloud/inventory.ini cluster.yml --become --become-user=root


## 2. Http доступ к web интерфейсу grafana.



## 3. Дашборды в grafana отображающие состояние Kubernetes кластера.



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

        root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl apply -f ./test-nginx.yaml
        pod/test-nginx-diplom created

      
      
        root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get pods
        NAME                READY   STATUS              RESTARTS   AGE
        test-nginx-diplom   0/1     ContainerCreating   0          9s
        root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get pods
        NAME                READY   STATUS    RESTARTS   AGE
        test-nginx-diplom   1/1     Running   0          12s

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
 
 



