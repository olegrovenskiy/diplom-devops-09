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




