# 1

проверить доступ в grafana и предоставить доступ туда

http://51.250.3.203:3000/?orgId=1 с логином/паролем admin / admin123

# 2  и 3

убрать ключи из .gitlab-ci.yaml

настроить сборку при коммите в любую ветку, а не только main. 

Выставление тега по имени ветки выполните по желанию.

Поправил .gitlab-ci.yml

        variables:
         image_tag: ${CI_COMMIT_TAG}

        stages:
          - release
          - build-exept-tag
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
            - docker login --username $USER --password $PASS cr.yandex
            - docker push cr.yandex/crpafmldih9te92441ft/app-nginx:${CI_COMMIT_BRANCH}


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
            - docker login --username oauth $USER $PASS cr.yandex
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

Результат

![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/branch.png)


#   4

в задании не требуется использование ALB, однако я настоятельно рекомендую вам попробовать использовать Ingress. 
Это позволит вам разобраться с технологией, которая используется почти во всех реальных кластерах kubernetes. Тогда вам не придётся с нуля разбираться с Ingress’ом, когда вы начнёте использовать kubernetes на работе. Но всё-таки выполнение этого пункта оставляю на ваше усмотрение.

