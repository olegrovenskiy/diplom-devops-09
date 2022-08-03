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
  root@mck-diplom-k8s-cp1:~/diplom/test-nginx#
  root@mck-diplom-k8s-cp1:~/diplom/test-nginx#
  root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get pods
  NAME                READY   STATUS              RESTARTS   AGE
  test-nginx-diplom   0/1     ContainerCreating   0          9s
  root@mck-diplom-k8s-cp1:~/diplom/test-nginx# kubectl get pods
  NAME                READY   STATUS    RESTARTS   AGE
  test-nginx-diplom   1/1     Running   0          12s

Под  запустился, образ успешно скачен, значить можно готовить деплоймент

