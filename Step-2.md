# Создание Kubernetes кластера

##  1. Работоспособный Kubernetes кластер.

С помощью Terraform подготовлено 3 ВМ. Пример изменённого файла 

main.tf :

    terraform {
      required_providers {
        yandex = {
          source = "yandex-cloud/yandex"
        }
      }
      required_version = ">= 0.13"


      backend "s3" {
        endpoint   = "storage.yandexcloud.net"
        bucket     = "netology-terraform-or"
        region     = "ru-central1"
        key        = "diplom.tfstate"
        access_key = "KEY"
        secret_key = "KEY"

        skip_region_validation      = true
        skip_credentials_validation = true
      }
    }

    provider "yandex" {
      token     = "TOKEN"
      cloud_id  = "cloud-orovenskiy"
      folder_id = "b1giu86qs432cv8j7c9p"
      zone      = "ru-central1-a"
    }

    resource "yandex_vpc_network" "network-diplom" {
      name = "diplom"
    }


    resource "yandex_vpc_subnet" "network-stage1" {
      name           = "stage-1"
      zone           = "ru-central1-a"
      network_id     = yandex_vpc_network.network-diplom.id
      v4_cidr_blocks = ["10.10.10.0/24"]

    }

    resource "yandex_vpc_subnet" "network-stage2" {
      name           = "stage-2"
      zone           = "ru-central1-b"
      network_id     = yandex_vpc_network.network-diplom.id
      v4_cidr_blocks = ["10.10.20.0/24"]

    }

    resource "yandex_vpc_subnet" "network-stage3" {
      name           = "stage-3"
      zone           = "ru-central1-c"
      network_id     = yandex_vpc_network.network-diplom.id
      v4_cidr_blocks = ["10.10.30.0/24"]

    }


    resource "yandex_compute_instance" "vm-1" {
      name = "cp-1"
      zone = "ru-central1-a"

      resources {
        cores  = 4
        memory = 4
      }

      boot_disk {
        initialize_params {
          image_id = "fd8mn5e1cksb3s1pcq12"
          size     = "100"
        }



      }

      network_interface {
        subnet_id = yandex_vpc_subnet.network-stage1.id
        nat       = true
        ip_address = "10.10.10.10"
      }

      metadata = {
        user-data = "${file("meta.txt")}"
      }

    }



    resource "yandex_compute_instance" "vm-2" {
      name = "wn-1"
      zone = "ru-central1-b"

      resources {
        cores  = 4
        memory = 4
      }

      boot_disk {
        initialize_params {
          image_id = "fd8mn5e1cksb3s1pcq12"
          size     = "100"
        }
      }

      network_interface {
        subnet_id = yandex_vpc_subnet.network-stage2.id
        nat       = true
        ip_address = "10.10.20.10"
      }

      metadata = {
        user-data = "${file("meta.txt")}"
      }

    }

    resource "yandex_compute_instance" "vm-3" {
      name = "wn-2"
      zone = "ru-central1-c"

      resources {
        cores  = 4
        memory = 4
      }

      boot_disk {
        initialize_params {
          image_id = "fd8mn5e1cksb3s1pcq12"
          size     = "100"
        }
      }

      network_interface {
        subnet_id = yandex_vpc_subnet.network-stage3.id
        nat       = true
        ip_address = "10.10.30.10"
      }

      metadata = {
        user-data = "${file("meta.txt")}"
      }

    }

Кластер разворачивал с помощью  ansible конфигурации и Kubespray
Пример

inventory.ini :

    oleg@mck-devops-tools:~/kubespray/inventory/yandexcloud$ cat inventory.ini
    # ## Configure 'ip' variable to bind kubernetes services on a
    # ## different ip than the default iface
    # ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.


    [all]
    mck-diplom-k8s-cp1 ansible_host=51.250.3.203  ip=10.10.10.10 etcd_member_name=etcd1
    mck-diplom-k8s-wn1 ansible_host=51.250.103.220  ip=10.10.20.10
    mck-diplom-k8s-wn2 ansible_host=51.250.44.45  ip=10.10.30.10


    # ## configure a bastion host if your nodes are not directly reachable
    # [bastion]
    # bastion ansible_host=x.x.x.x ansible_user=some_user

    [kube_control_plane]
    mck-diplom-k8s-cp1


    [etcd]
    mck-diplom-k8s-cp1


    [kube_node]
    mck-diplom-k8s-wn1
    mck-diplom-k8s-wn2


    [calico_rr]

    [k8s_cluster:children]
    kube_control_plane
    kube_node
    calico_rr

    [all:vars]
    ansible_sudo_pass=PASSWORD
    ansible_user=oleg
    ansible_password=PASSWORD
    oleg@mck-devops-tools:~/kubespray/inventory/yandexcloud$


Ноды запущены и в работе:

    root@mck-diplom-k8s-cp1:~/.docker# kubectl get nodes
    NAME                 STATUS   ROLES                  AGE   VERSION
    mck-diplom-k8s-cp1   Ready    control-plane,master   18h   v1.23.5
    mck-diplom-k8s-wn1   Ready    <none>                 18h   v1.23.5
    mck-diplom-k8s-wn2   Ready    <none>                 18h   v1.23.5
    root@mck-diplom-k8s-cp1:~/.docker#


В файле ~/.kube/config находятся данные для доступа к кластеру.

    root@mck-diplom-k8s-cp1:~# cat ~/.kube/config
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data:     
        server: https://127.0.0.1:6443
      name: k8s.mgc.local
    contexts:
    - context:
        cluster: k8s.mgc.local
        user: kubernetes-admin
      name: kubernetes-admin@k8s.mgc.local
    current-context: kubernetes-admin@k8s.mgc.local
    kind: Config
    preferences: {}
    users:
    - name: kubernetes-admin
      user:
        client-certificate-data:     
        client-key-data: 
    root@mck-diplom-k8s-cp1:~#


Команда kubectl get pods --all-namespaces отрабатывает без ошибок.

    oleg@mck-diplom-k8s-cp1:~$ sudo kubectl get pods --all-namespaces
    [sudo] password for oleg:
    NAMESPACE       NAME                                         READY   STATUS    RESTARTS   AGE
    ingress-nginx   ingress-nginx-controller-98sr2               1/1     Running   0          119s
    ingress-nginx   ingress-nginx-controller-zqx4q               1/1     Running   0          119s
    kube-system     calico-kube-controllers-75fcdd655b-c8qdf     1/1     Running   0          2m18s
    kube-system     calico-node-6qtjd                            1/1     Running   0          2m47s
    kube-system     calico-node-lcdkm                            1/1     Running   0          2m47s
    kube-system     calico-node-zb9pd                            1/1     Running   0          2m47s
    kube-system     coredns-76b4fb4578-jg566                     1/1     Running   0          97s
    kube-system     coredns-76b4fb4578-z92bm                     1/1     Running   0          87s
    kube-system     dns-autoscaler-7979fb6659-f9kq5              1/1     Running   0          90s
    kube-system     kube-apiserver-mck-diplom-k8s-cp1            1/1     Running   0          4m23s
    kube-system     kube-controller-manager-mck-diplom-k8s-cp1   1/1     Running   1          4m20s
    kube-system     kube-proxy-k8fnv                             1/1     Running   0          3m12s
    kube-system     kube-proxy-q8fq9                             1/1     Running   0          3m12s
    kube-system     kube-proxy-xw4br                             1/1     Running   0          3m12s
    kube-system     kube-scheduler-mck-diplom-k8s-cp1            1/1     Running   1          4m20s
    kube-system     nginx-proxy-mck-diplom-k8s-wn1               1/1     Running   0          3m19s
    kube-system     nginx-proxy-mck-diplom-k8s-wn2               1/1     Running   0          3m19s
    kube-system     nodelocaldns-krbch                           1/1     Running   0          90s
    kube-system     nodelocaldns-lnlch                           1/1     Running   0          90s
    kube-system     nodelocaldns-w6qs7                           1/1     Running   0          90s
    oleg@mck-diplom-k8s-cp1:~$




