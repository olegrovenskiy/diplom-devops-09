# Создание тестового приложения

##  1. Подготовка файлов

Конфиг nginx, Докерфайл, и файл картинки в репозитории https://github.com/olegrovenskiy/diplom-nginx-test-app

Приложение отдаёт картинку (тестировалось уже в контейнере)

Проверка curl

    oleg@mck-devops-tools:/etc/nginx$ curl localhost/images/image.webp
    Warning: Binary output can mess up your terminal. Use "--output -" to tell
    Warning: curl to output it to your terminal anyway, or consider "--output
    Warning: <FILE>" to save to a file.
    oleg@mck-devops-tools:/etc/nginx$
    
  И из веб браузера
  
  ![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/appl.png)
  
  ##    Регистр для докер образа
  
  В качестве регистра использован Yandex Container Registry, созданный также с помощью terraform, для чего 
  в main.tf был добавлен следующий код :
  
          resource "yandex_container_registry" "my-reg" {
          name = "diplom-registry"
          folder_id = "b1giu86qs432cv8j7c9p"
          labels = {
          my-label = "reg-diplom"
          }
        }
  ##    Push образа
  
  Далее на основе Докерфайла был запущен контейнер, скрины проверки приложения представлены авше.
  Из рабочего контейнера был создан новый образ :
  
        root@mck-diplom-k8s-cp1:~/diplom# docker commit -m "custom_config" -a "olegrovenskiy" a9ccf1873582 nginx:diplom
        sha256:91e1aba72a982a626f7992847f12ca6c09ebdb0b3710304f17d86be3e61f3677
        root@mck-diplom-k8s-cp1:~/diplom#
        
        root@mck-diplom-k8s-cp1:~/diplom# docker tag nginx:diplom cr.yandex/crpafmldih9te92441ft/nginx:diplom
        root@mck-diplom-k8s-cp1:~/diplom# docker images
        REPOSITORY                                             TAG       IMAGE ID       CREATED         SIZE
        cr.yandex/crpafmldih9te92441ft/nginx                   diplom    91e1aba72a98   7 minutes ago   142MB
        
        
После чего образ был запушен в репозиторий

        root@mck-diplom-k8s-cp1:~/.docker# docker push cr.yandex/crpafmldih9te92441ft/nginx:diplom
        The push refers to repository [cr.yandex/crpafmldih9te92441ft/nginx]
        0c0fb8027268: Pushed
        3d655bf6c956: Pushed
        607cb68689ce: Pushed
        5ff3c9ddb7d8: Pushed
        237325c8292d: Pushed
        b539cf60d7bb: Pushed
        bdc7a32279cc: Pushed
        f91d0987b144: Pushed
        3a89c8160a43: Pushed
        e3257a399753: Pushed
        92a4e8a3140f: Pushed
        diplom: digest: sha256:38140d4eb46712f1211ea603b5ea6c027929cdd2421006a169f4c28b44182611 size: 2607
        root@mck-diplom-k8s-cp1:~/.docker#

 ![sonar1](https://github.com/olegrovenskiy/diplom-devops-09/blob/main/registry.png)
 
 Для работы использовал документацию яндекс - https://cloud.yandex.ru/docs/container-registry/quickstart/
 
 


  
  
