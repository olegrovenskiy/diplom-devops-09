# Подготовка cистемы мониторинга и деплой приложения

Ожидаемый результат:

Git репозиторий с конфигурационными файлами для настройки Kubernetes.

https://github.com/olegrovenskiy/diplom-k8s-config/tree/master

запуск плейбука

 ansible-playbook -i inventory/yandexcloud/inventory.ini cluster.yml --become --become-user=root


Http доступ к web интерфейсу grafana.
Дашборды в grafana отображающие состояние Kubernetes кластера.
Http доступ к тестовому приложению.

