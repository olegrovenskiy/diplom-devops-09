# Установка и настройка CI/CD

Цель:

Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
Автоматический деплой нового docker образа.

Для выполнения использовал:

1. CI/CD на основе GitLab
2. Проект https://gitlab.com/olegrovenskiy/diplom-test
3. Регистри для докер образов на ЯндексКлауд
4. Кластер k8s добавлен в инфраструктуру Гитлаба, агент в статусе коннект

![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/agent.png)

Тестовый деплой

![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/build.png)

Тестовый билд

![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/deploy.png)

Базовый рабочий .gitlab-ci.yml file

    stages:
      - build
      - deploy

    build:
      stage: build
      image: docker:latest
      services:
        - docker:18.09.7-dind
      script:
        - docker build . --tag cr.yandex/crpafmldih9te92441ft/app-nginx
        - docker login --username oauth --password PASSWORD cr.yandex
        - docker push cr.yandex/crpafmldih9te92441ft/app-nginx

    deploy:
      stage: deploy
      image:
        name: bitnami/kubectl:latest
        entrypoint: ['']
      script:
        - kubectl config get-contexts
        - kubectl config use-context olegrovenskiy/diplom-test:agent-new
        - git pull https://gitlab.com/olegrovenskiy/diplom-test.git
        - kubectl apply -f ./deploy-test-app.yaml
        - kubectl get pods


По умолчанию при комите в ветке main происходит build с пушем нового образа, и deploy 
Использованно: 

документация https://docs.gitlab.com/ee/user/clusters/agent/ci_cd_workflow.html#update-your-gitlab-ciyml-file-to-run-kubectl-commands
GitLab CI/CD SaaS был подготовлен ранее на блоен CI/CD
Дополнительно https://stackoverflow.com/questions/51196435/gitlab-ci-docker-command-not-found

Проверка работы приложения

http://51.250.3.203:32757/images/1.jpg

![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/aplicat.png)



- добавление 
        when: manual
в stage deploy .gitlab-ci.yml file обеспечивает только прохождение автоматичаской сборки образа при комитах в репозитории, что соответствует пункту задания.

Для выполнения условий:

При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистр, а также деплой соответствующего Docker образа в кластер Kubernetes.

были скорректированы файлы:

1. .gitlab-ci.yml

        variables:
         image_tag: ${CI_COMMIT_TAG}

        stages:
          - release
          - build-main
          - build-tag
          - deploy

        release_job:
          stage: release
          image: registry.gitlab.com/gitlab-org/release-cli:latest
          rules:
            - if: $CI_COMMIT_TAG                  # Run this job when a tag is created manually
          script:
            - echo "Running the release job."
          release:
            tag_name: $CI_COMMIT_TAG
            name: 'Release $CI_COMMIT_TAG'
            description: 'Release created using the release-cli.'  


        build-main:
          stage: build-main
          image: docker:latest
          services:
            - docker:18.09.7-dind

          script:
            - docker build . --tag app-nginx
            - docker tag app-nginx cr.yandex/crpafmldih9te92441ft/app-nginx:latest
            - docker login --username oauth --password AQAAAABapegEAATuwQiNejL4zEschjtrCjlWlqU cr.yandex
            - docker push cr.yandex/crpafmldih9te92441ft/app-nginx:latest
            - echo "MAIN-MAIN"

          only:
            - main



        build-tag:
          stage: build-tag
          image: docker:latest
          services:
            - docker:18.09.7-dind
          rules:
            - if: $CI_COMMIT_TAG

          script:
            - docker build . --tag app-nginx
            - docker tag app-nginx cr.yandex/crpafmldih9te92441ft/app-nginx:${CI_COMMIT_TAG}
            - docker login --username oauth --password AQAAAABapegEAATuwQiNejL4zEschjtrCjlWlqU cr.yandex
            - docker push cr.yandex/crpafmldih9te92441ft/app-nginx:${CI_COMMIT_TAG}
            - echo "TAG-TAG"



        deploy:
          stage: deploy
          image:
            name: bitnami/kubectl:latest
            entrypoint: ['']
          rules:
            - if: $CI_COMMIT_TAG
          script:
            - kubectl config get-contexts
            - kubectl config use-context olegrovenskiy/diplom-test:agent-new
            - git pull https://gitlab.com/olegrovenskiy/diplom-test.git
            - sed -i "s/<version>/${CI_COMMIT_TAG}/g" deploy-test-app.yaml
            - kubectl apply -f ./deploy-test-app.yaml
            - kubectl get pods


2. И кубернетес манифест deploy-test-app.yam

                  apiVersion: apps/v1
                  kind: Deployment
                  metadata:
                    labels:
                      app: app-nginx-diplom
                    name: app-nginx-diplom
                    namespace: default
                  spec:
                    replicas: 3
                    selector:
                      matchLabels:
                        app: app-nginx-diplom
                    template:
                       metadata:
                         labels:
                           app: app-nginx-diplom
                       spec:
                         containers:
                           - image: cr.yandex/crpafmldih9te92441ft/app-nginx:<version>
                             name: app-nginx-diplom
                         imagePullSecrets:
                           - name: regcred
        ---

                  apiVersion: v1
                  kind: Service
                  metadata:
                    name: app-nginx-diplom
                    namespace: default
                  spec:
                    ports:
                      - name: web
                        port: 8089
                        targetPort: 80
                        nodePort: 32757
                    selector:
                      app: app-nginx-diplom
                    type: NodePort

После чего при создании тега стал пушиться образ с тегом 

![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/tag.png)

И успешно запускаются поды с данным образом

            root@mck-diplom-k8s-cp1:~/diplom/diplom-test# kubectl get pods
            NAME                                 READY   STATUS    RESTARTS       AGE
            app-nginx-diplom-98f78d9bf-dbpnt     1/1     Running   0              10m
            app-nginx-diplom-98f78d9bf-fzbq8     1/1     Running   0              10m
            app-nginx-diplom-98f78d9bf-qmcss     1/1     Running   0              10m
            gitlab-runner-7589897d7-ds8h2        1/1     Running   20 (29h ago)   33h
            test-nginx-diplom-578479f76f-fmj8m   1/1     Running   0              2d13h
            test-nginx-diplom-578479f76f-h42dg   1/1     Running   0              2d13h
            test-nginx-diplom-578479f76f-ttkdt   1/1     Running   0              2d13h
            root@mck-diplom-k8s-cp1:~/diplom/diplom-test#
            root@mck-diplom-k8s-cp1:~/diplom/diplom-test# kubectl get pod app-nginx-diplom-98f78d9bf-dbpnt -o json
            ....
            "spec": {
                    "containers": [
                        {
                            "image": "cr.yandex/crpafmldih9te92441ft/app-nginx:v1.0.0",
                            "imagePullPolicy": "Always",
                            "name": "app-nginx-diplom",



Для проброса переменных использовал : https://stackoverflow.com/questions/56705484/how-to-pass-gitlab-ci-cd-variables-to-kubernetesaks-deployment-yaml







