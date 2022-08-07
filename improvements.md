# 1

проверить доступ в grafana и предоставить доступ туда

http://51.250.3.203:3000/?orgId=1 с логином/паролем admin / admin123

# 2


#3  настроить сборку при коммите в любую ветку, а не только main. 

Выставление тега по имени ветки выполните по желанию.

Поправил скрипт

build-exept-tag:
  stage: build-exept-tag
  image: docker:latest
  services:
    - docker:18.09.7-dind
  rules:
    - if: $CI_COMMIT_BRANCH 
  script:
    - docker build . --tag app-nginx
    - docker tag app-nginx cr.yandex/crpafmldih9te92441ft/app-nginx:${CI_COMMIT_BRANCH}
    - docker login --username oauth --password AQAAAABapegEAATuwQiNejL4zEschjtrCjlWlqU cr.yandex
    - docker push cr.yandex/crpafmldih9te92441ft/app-nginx:${CI_COMMIT_BRANCH}
    - echo "MAIN-MAIN"

Результат

![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/branch.png)

