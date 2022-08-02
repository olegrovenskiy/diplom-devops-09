# Создание облачной инфраструктуры

Ожидаемые результаты:

Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий.
Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.


##  1. Подготовка Terraform

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
        access_key = "ACCESS-KEY"
        secret_key = "SECRET-KEY"

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


Создан workspace  - stage

    C:\Oleg\HW-Terraform>terraform workspace new stage
    Created and switched to workspace "stage"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.

В качестве backend для Terraform использовал S3 bucket в созданном ЯО аккаунте

Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

    Initializing provider plugins...
    - Reusing previous version of yandex-cloud/yandex from the dependency lock file
    - Using previously-installed yandex-cloud/yandex v0.75.0

    Terraform has been successfully initialized!

VPC создано

Команды terraform destroy и terraform apply проходят без дополнительных ручных действий.

    C:\Oleg\HW-Terraform>terraform apply

    Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
    following symbols:
      + create

    Terraform will perform the following actions:

      # yandex_vpc_network.network-diplom will be created
      + resource "yandex_vpc_network" "network-diplom" {
          + created_at                = (known after apply)
          + default_security_group_id = (known after apply)
          + folder_id                 = (known after apply)
          + id                        = (known after apply)
          + labels                    = (known after apply)
          + name                      = "diplom"
          + subnet_ids                = (known after apply)
        }

      # yandex_vpc_subnet.network-stage1 will be created
      + resource "yandex_vpc_subnet" "network-stage1" {
          + created_at     = (known after apply)
          + folder_id      = (known after apply)
          + id             = (known after apply)
          + labels         = (known after apply)
          + name           = "stage-1"
          + network_id     = (known after apply)
          + v4_cidr_blocks = [
              + "10.10.10.0/24",
            ]
          + v6_cidr_blocks = (known after apply)
          + zone           = "ru-central1-a"
        }

      # yandex_vpc_subnet.network-stage2 will be created
      + resource "yandex_vpc_subnet" "network-stage2" {
          + created_at     = (known after apply)
          + folder_id      = (known after apply)
          + id             = (known after apply)
          + labels         = (known after apply)
          + name           = "stage-2"
          + network_id     = (known after apply)
          + v4_cidr_blocks = [
              + "10.10.20.0/24",
            ]
          + v6_cidr_blocks = (known after apply)
          + zone           = "ru-central1-b"
        }

      # yandex_vpc_subnet.network-stage3 will be created
      + resource "yandex_vpc_subnet" "network-stage3" {
          + created_at     = (known after apply)
          + folder_id      = (known after apply)
          + id             = (known after apply)
          + labels         = (known after apply)
          + name           = "stage-3"
          + network_id     = (known after apply)
          + v4_cidr_blocks = [
              + "10.10.30.0/24",
            ]
          + v6_cidr_blocks = (known after apply)
          + zone           = "ru-central1-c"
        }

    Plan: 4 to add, 0 to change, 0 to destroy.

    Do you want to perform these actions in workspace "stage"?
      Terraform will perform the actions described above.
      Only 'yes' will be accepted to approve.

      Enter a value: yes

    yandex_vpc_network.network-diplom: Creating...
    yandex_vpc_network.network-diplom: Creation complete after 1s [id=enpva44samt1m00709jt]
    yandex_vpc_subnet.network-stage3: Creating...
    yandex_vpc_subnet.network-stage1: Creating...
    yandex_vpc_subnet.network-stage2: Creating...
    yandex_vpc_subnet.network-stage3: Creation complete after 1s [id=b0c695k663n6sq03reao]
    yandex_vpc_subnet.network-stage1: Creation complete after 2s [id=e9bpt105trjap0soc5sp]
    yandex_vpc_subnet.network-stage2: Creation complete after 2s [id=e2ldmtbv0v5mnn5g5v9i]

    Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

    C:\Oleg\HW-Terraform>


