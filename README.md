# «Вычислительные мощности. Балансировщики нагрузки» - Абрамов Сергей 

### Подготовка к выполнению задания

1. Домашнее задание состоит из обязательной части, которую нужно выполнить на провайдере Yandex Cloud, и дополнительной части в AWS (выполняется по желанию). 
2. Все домашние задания в блоке 15 связаны друг с другом и в конце представляют пример законченной инфраструктуры.  
3. Все задания нужно выполнить с помощью Terraform. Результатом выполненного домашнего задания будет код в репозитории. 
4. Перед началом работы настройте доступ к облачным ресурсам из Terraform, используя материалы прошлых лекций и домашних заданий.

---
## Задание 1. Yandex Cloud 

**Что нужно сделать**

1. Создать бакет Object Storage и разместить в нём файл с картинкой:

 - Создать бакет в Object Storage с произвольным именем (например, _имя_студента_дата_).
 - Положить в бакет файл с картинкой.
 - Сделать файл доступным из интернета.

 Подготовил пайплайн и применил:

```
serg@debian:~/git/cloude-15-2$ terraform apply
yandex_storage_object.picture: Refreshing state... [id=picture.jpg]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_storage_bucket.my_bucket will be created
  + resource "yandex_storage_bucket" "my_bucket" {
      + access_key            = (known after apply)
      + acl                   = "public-read"
      + bucket                = "smabramov-2025-11-01"
      + bucket_domain_name    = (known after apply)
      + default_storage_class = (known after apply)
      + folder_id             = (known after apply)
      + force_destroy         = false
      + id                    = (known after apply)
      + secret_key            = (sensitive value)
      + website_domain        = (known after apply)
      + website_endpoint      = (known after apply)
    }

  # yandex_storage_object.picture will be created
  + resource "yandex_storage_object" "picture" {
      + access_key   = (known after apply)
      + acl          = "public-read"
      + bucket       = "smabramov-2025-11-01"
      + content_type = (known after apply)
      + id           = (known after apply)
      + key          = "picture.jpg"
      + secret_key   = (sensitive value)
      + source       = "./picture.jpg"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_storage_object.picture: Creating...
yandex_storage_bucket.my_bucket: Creating...
yandex_storage_object.picture: Creation complete after 0s [id=picture.jpg]
yandex_storage_bucket.my_bucket: Creation complete after 2s [id=smabramov-2025-11-01]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```
![picture]()

2. Создать группу ВМ в public подсети фиксированного размера с шаблоном LAMP и веб-страницей, содержащей ссылку на картинку из бакета:

 - Создать Instance Group с тремя ВМ и шаблоном LAMP. Для LAMP рекомендуется использовать `image_id = fd827b91d99psvq5fjit`.
 - Для создания стартовой веб-страницы рекомендуется использовать раздел `user_data` в [meta_data](https://cloud.yandex.ru/docs/compute/concepts/vm-metadata).
 - Разместить в стартовой веб-странице шаблонной ВМ ссылку на картинку из бакета.
 - Настроить проверку состояния ВМ.

```
serg@debian:~/git/cloude-15-2$ terraform apply
yandex_storage_object.picture: Refreshing state... [id=picture.jpg]
yandex_storage_bucket.my_bucket: Refreshing state... [id=smabramov-2025-11-01]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
 + create

Terraform will perform the following actions:

 # yandex_compute_instance_group.ig-1 will be created
 + resource "yandex_compute_instance_group" "ig-1" {
     + created_at          = (known after apply)
     + deletion_protection = false
     + folder_id           = "b1g8381i07tsfq06pnmc"
     + id                  = (known after apply)
     + instances           = (known after apply)
     + name                = "fixed-ig-with-balancer"
     + service_account_id  = (known after apply)
     + status              = (known after apply)

     + allocation_policy {
         + zones = [
             + "ru-central1-a",
           ]
       }

     + deploy_policy {
         + max_creating     = 0
         + max_deleting     = 0
         + max_expansion    = 0
         + max_unavailable  = 1
         + startup_duration = 0
         + strategy         = (known after apply)
       }

     + instance_template {
         + labels      = (known after apply)
         + metadata    = {
             + "ssh-keys"  = "debian:~/.ssh/id_rsa.pub"
             + "user-data" = <<-EOT
                   #cloud-config
                   
                   write_files:
                   - content: |
                       <html>
                       <body>
                       <p><?php echo "hostname is:".gethostname(); ?></p>
                       <p>
                       <img src="https://storage.yandexcloud.net/smabramov-2025-11-01/picture.jpg" alt="picture" width="500" height="600">
                       </p>
                       </body>    
                       </html>
                     path: /var/www/html/index.html
                     defer: true
                   - content: |
                       <html>
                       <body>
                       <p><?php echo "hostname is:".gethostname(); ?></p>
                       <p>
                       <img src="https://storage.yandexcloud.net/smabramov-2025-11-01/picture.jpg" alt="icture" width="500" height="600">
                       </p>
                       </body>    
                       </html>
                     path: /var/www/html/index.php
                     defer: true
               EOT
           }
         + platform_id = "standard-v1"

         + boot_disk {
             + device_name = (known after apply)
             + mode        = "READ_WRITE"

             + initialize_params {
                 + image_id    = "fd827b91d99psvq5fjit"
                 + size        = (known after apply)
                 + snapshot_id = (known after apply)
                 + type        = "network-hdd"
               }
           }

         + metadata_options (known after apply)

         + network_interface {
             + ip_address   = (known after apply)
             + ipv4         = true
             + ipv6         = (known after apply)
             + ipv6_address = (known after apply)
             + nat          = (known after apply)
             + network_id   = (known after apply)
             + subnet_ids   = (known after apply)
           }

         + resources {
             + core_fraction = 20
             + cores         = 2
             + memory        = 2
           }

         + scheduling_policy (known after apply)
       }

     + load_balancer {
         + status_message           = (known after apply)
         + target_group_description = "Целевая группа Network Load Balancer"
         + target_group_id          = (known after apply)
         + target_group_name        = "target-group"
       }

     + scale_policy {
         + fixed_scale {
             + size = 3
           }
       }
   }

 # yandex_iam_service_account.sa-gvm will be created
 + resource "yandex_iam_service_account" "sa-gvm" {
     + created_at  = (known after apply)
     + description = "Сервисный аккаунт для управления группой ВМ."
     + folder_id   = (known after apply)
     + id          = (known after apply)
     + name        = "sa-gvm"
   }

 # yandex_resourcemanager_folder_iam_member.editor will be created
 + resource "yandex_resourcemanager_folder_iam_member" "editor" {
     + folder_id = "b1g8381i07tsfq06pnmc"
     + id        = (known after apply)
     + member    = (known after apply)
     + role      = "editor"
   }

 # yandex_vpc_network.VPC will be created
 + resource "yandex_vpc_network" "VPC" {
     + created_at                = (known after apply)
     + default_security_group_id = (known after apply)
     + folder_id                 = (known after apply)
     + id                        = (known after apply)
     + labels                    = (known after apply)
     + name                      = "VPC"
     + subnet_ids                = (known after apply)
   }

 # yandex_vpc_subnet.public will be created
 + resource "yandex_vpc_subnet" "public" {
     + created_at     = (known after apply)
     + folder_id      = (known after apply)
     + id             = (known after apply)
     + labels         = (known after apply)
     + name           = "public"
     + network_id     = (known after apply)
     + v4_cidr_blocks = [
         + "192.168.10.0/24",
       ]
     + v6_cidr_blocks = (known after apply)
     + zone           = "ru-central1-a"
   }

Plan: 5 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
 Terraform will perform the actions described above.
 Only 'yes' will be accepted to approve.

 Enter a value: yes

yandex_vpc_network.VPC: Creating...
yandex_iam_service_account.sa-gvm: Creating...
yandex_iam_service_account.sa-gvm: Creation complete after 3s [id=ajefsg2lh2irq8rs10g2]
yandex_resourcemanager_folder_iam_member.editor: Creating...
yandex_vpc_network.VPC: Creation complete after 3s [id=enpl0qdfskoq82n507t0]
yandex_vpc_subnet.public: Creating...
yandex_vpc_subnet.public: Creation complete after 1s [id=e9bu1r35uukjqin5p6es]
yandex_compute_instance_group.ig-1: Creating...
yandex_resourcemanager_folder_iam_member.editor: Creation complete after 3s [id=b1gpta86451pk7tseq2b/editor/serviceAccount:ajei98u1s9ap4tn8bdg1]
yandex_compute_instance_group.ig-1: Still creating... [10s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [20s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [30s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [40s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [50s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m0s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m10s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m20s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m30s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m40s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m50s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m0s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m10s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m20s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m30s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m40s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m50s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [3m0s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [3m10s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [3m20s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [3m31s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [3m41s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [3m51s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [4m1s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [4m11s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [4m21s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [4m31s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [4m41s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [4m51s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [5m1s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [5m11s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [5m21s elapsed]
yandex_compute_instance_group.ig-1: Creation complete after 5m29s [id=enp1hqcennkafeb9f5t7]

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.
```
![vm]()

 
3. Подключить группу к сетевому балансировщику:

 - Создать сетевой балансировщик.
 - Проверить работоспособность, удалив одну или несколько ВМ.

 ```
 serg@debian:~/git/cloude-15-2$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
 + create

Terraform will perform the following actions:

 # yandex_compute_instance_group.ig-1 will be created
 + resource "yandex_compute_instance_group" "ig-1" {
     + created_at          = (known after apply)
     + deletion_protection = false
     + folder_id           = "b1g8381i07tsfq06pnmc"
     + id                  = (known after apply)
     + instances           = (known after apply)
     + name                = "fixed-ig-with-balancer"
     + service_account_id  = (known after apply)
     + status              = (known after apply)

     + allocation_policy {
         + zones = [
             + "ru-central1-a",
           ]
       }

     + deploy_policy {
         + max_creating     = 0
         + max_deleting     = 0
         + max_expansion    = 0
         + max_unavailable  = 1
         + startup_duration = 0
         + strategy         = (known after apply)
       }

     + health_check {
         + healthy_threshold   = 2
         + interval            = 30
         + timeout             = 10
         + unhealthy_threshold = 2

         + tcp_options {
             + port = 80
           }
       }

     + instance_template {
         + labels      = (known after apply)
         + metadata    = (known after apply)
         + platform_id = "standard-v1"

         + boot_disk {
             + device_name = (known after apply)
             + mode        = "READ_WRITE"

             + initialize_params {
                 + image_id    = "fd827b91d99psvq5fjit"
                 + size        = (known after apply)
                 + snapshot_id = (known after apply)
                 + type        = "network-hdd"
               }
           }

         + metadata_options (known after apply)

         + network_interface {
             + ip_address   = (known after apply)
             + ipv4         = true
             + ipv6         = (known after apply)
             + ipv6_address = (known after apply)
             + nat          = (known after apply)
             + network_id   = (known after apply)
             + subnet_ids   = (known after apply)
           }

         + resources {
             + core_fraction = 20
             + cores         = 2
             + memory        = 2
           }

         + scheduling_policy (known after apply)
       }

     + load_balancer {
         + status_message           = (known after apply)
         + target_group_description = "Целевая группа Network Load Balancer"
         + target_group_id          = (known after apply)
         + target_group_name        = "target-group"
       }

     + scale_policy {
         + fixed_scale {
             + size = 3
           }
       }
   }

 # yandex_iam_service_account.sa-gvm will be created
 + resource "yandex_iam_service_account" "sa-gvm" {
     + created_at = (known after apply)
     + folder_id  = (known after apply)
     + id         = (known after apply)
     + name       = "sa-gvm"
   }

 # yandex_iam_service_account.service will be created
 + resource "yandex_iam_service_account" "service" {
     + created_at = (known after apply)
     + folder_id  = "b1g8381i07tsfq06pnmc"
     + id         = (known after apply)
     + name       = "bucket-sa"
   }

 # yandex_iam_service_account_static_access_key.sa-static-key will be created
 + resource "yandex_iam_service_account_static_access_key" "sa-static-key" {
     + access_key                   = (known after apply)
     + created_at                   = (known after apply)
     + description                  = "static access key for object storage"
     + encrypted_secret_key         = (known after apply)
     + id                           = (known after apply)
     + key_fingerprint              = (known after apply)
     + output_to_lockbox_version_id = (known after apply)
     + secret_key                   = (sensitive value)
     + service_account_id           = (known after apply)
   }

 # yandex_lb_network_load_balancer.lb-1 will be created
 + resource "yandex_lb_network_load_balancer" "lb-1" {
     + created_at          = (known after apply)
     + deletion_protection = (known after apply)
     + folder_id           = (known after apply)
     + id                  = (known after apply)
     + name                = "network-load-balancer-1"
     + region_id           = (known after apply)
     + type                = "external"

     + attached_target_group {
         + target_group_id = (known after apply)

         + healthcheck {
             + healthy_threshold   = 5
             + interval            = 2
             + name                = "http"
             + timeout             = 1
             + unhealthy_threshold = 2

             + http_options {
                 + path = "/index.html"
                 + port = 80
               }
           }
       }

     + listener {
         + name        = "network-load-balancer-1-listener"
         + port        = 80
         + protocol    = (known after apply)
         + target_port = (known after apply)

         + external_address_spec {
             + address    = (known after apply)
             + ip_version = "ipv4"
           }
       }
   }

 # yandex_resourcemanager_folder_iam_member.bucket-editor will be created
 + resource "yandex_resourcemanager_folder_iam_member" "bucket-editor" {
     + folder_id = "b1g8381i07tsfq06pnmc"
     + id        = (known after apply)
     + member    = (known after apply)
     + role      = "storage.editor"
   }

 # yandex_resourcemanager_folder_iam_member.editor will be created
 + resource "yandex_resourcemanager_folder_iam_member" "editor" {
     + folder_id = "b1g8381i07tsfq06pnmc"
     + id        = (known after apply)
     + member    = (known after apply)
     + role      = "editor"
   }

 # yandex_storage_bucket.my_bucket will be created
 + resource "yandex_storage_bucket" "my_bucket" {
     + access_key            = (known after apply)
     + acl                   = "public-read"
     + bucket                = "smabramov-2025-11-01"
     + bucket_domain_name    = (known after apply)
     + default_storage_class = (known after apply)
     + folder_id             = (known after apply)
     + force_destroy         = false
     + id                    = (known after apply)
     + secret_key            = (sensitive value)
     + website_domain        = (known after apply)
     + website_endpoint      = (known after apply)

     + anonymous_access_flags (known after apply)

     + versioning (known after apply)
   }

 # yandex_storage_object.picture will be created
 + resource "yandex_storage_object" "picture" {
     + access_key   = (known after apply)
     + acl          = "public-read"
     + bucket       = "smabramov-2025-11-01"
     + content_type = (known after apply)
     + id           = (known after apply)
     + key          = "picture.jpg"
     + secret_key   = (sensitive value)
     + source       = "./picture.jpg"
   }

 # yandex_vpc_network.VPC will be created
 + resource "yandex_vpc_network" "VPC" {
     + created_at                = (known after apply)
     + default_security_group_id = (known after apply)
     + folder_id                 = (known after apply)
     + id                        = (known after apply)
     + labels                    = (known after apply)
     + name                      = "VPC"
     + subnet_ids                = (known after apply)
   }

 # yandex_vpc_subnet.public will be created
 + resource "yandex_vpc_subnet" "public" {
     + created_at     = (known after apply)
     + folder_id      = (known after apply)
     + id             = (known after apply)
     + labels         = (known after apply)
     + name           = "public"
     + network_id     = (known after apply)
     + v4_cidr_blocks = [
         + "192.168.10.0/24",
       ]
     + v6_cidr_blocks = (known after apply)
     + zone           = "ru-central1-a"
   }

Plan: 11 to add, 0 to change, 0 to destroy.

Changes to Outputs:
 + backet        = (known after apply)
 + lb_ip_address = [
     + (known after apply),
   ]
 + picture_key   = "picture.jpg"

Do you want to perform these actions?
 Terraform will perform the actions described above.
 Only 'yes' will be accepted to approve.

 Enter a value: yes

yandex_vpc_network.VPC: Creating...
yandex_iam_service_account.service: Creating...
yandex_iam_service_account.sa-gvm: Creating...
yandex_vpc_network.VPC: Creation complete after 2s [id=enpl0qdfskoq82n507t0]
yandex_vpc_subnet.public: Creating...
yandex_vpc_subnet.public: Creation complete after 1s [id=e9bu1r35uukjqin5p6es]
yandex_iam_service_account.service: Creation complete after 3s [id=aje7469jsdejkcq050nd]
yandex_resourcemanager_folder_iam_member.bucket-editor: Creating...
yandex_iam_service_account_static_access_key.sa-static-key: Creating...
yandex_iam_service_account_static_access_key.sa-static-key: Creation complete after 1s [id=ajeet42okglorodolnrb]
yandex_storage_bucket.my_bucket: Creating...
yandex_iam_service_account.sa-gvm: Creation complete after 5s [id=ajefsg2lh2irq8rs10g2]
yandex_resourcemanager_folder_iam_member.editor: Creating...
yandex_resourcemanager_folder_iam_member.bucket-editor: Creation complete after 3s [id=b1gpta86451pk7tseq2b/storage.editor/serviceAccount:aje7469jsdejkcq050nd]
yandex_storage_bucket.my_bucket: Creation complete after 4s [id=smabramov-2025-11-01]
yandex_storage_object.picture: Creating...
yandex_resourcemanager_folder_iam_member.editor: Creation complete after 4s [id=b1gpta86451pk7tseq2b/editor/serviceAccount:aje25rjo8erbbl0j445q]
yandex_storage_object.picture: Creation complete after 1s [id=picture.jpg]
yandex_compute_instance_group.ig-1: Creating...
yandex_compute_instance_group.ig-1: Still creating... [10s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [20s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [30s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [40s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [50s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m0s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m10s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m20s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m30s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m40s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [1m50s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m0s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m10s elapsed]
yandex_compute_instance_group.ig-1: Still creating... [2m20s elapsed]
yandex_compute_instance_group.ig-1: Creation complete after 2m26s [id=enpp5l2cr8o3luqfvtg3]
yandex_lb_network_load_balancer.lb-1: Creating...
yandex_lb_network_load_balancer.lb-1: Creation complete after 7s [id=enp1hqcennkafeb9f5t7]

Apply complete! Resources: 11 added, 0 changed, 0 destroyed.

Outputs:

backet = "smabramov-2025-11-01.storage.yandexcloud.net"
lb_ip_address = tolist([
 "158.160.135.179",
])
picture_key = "picture.jpg"
```
![balancer]()

![index]()

Удаляю две вм и проверяю снова:

![delete]()

![index]()

ВМ пресоздаются:

![recreate]()

Ссылка на манифесты - [ссылка]()

4. (дополнительно)* Создать Application Load Balancer с использованием Instance group и проверкой состояния.

Полезные документы:

- [Compute instance group](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/compute_instance_group).
- [Network Load Balancer](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/lb_network_load_balancer).
- [Группа ВМ с сетевым балансировщиком](https://cloud.yandex.ru/docs/compute/operations/instance-groups/create-with-balancer).

---
## Задание 2*. AWS (задание со звёздочкой)

Это необязательное задание. Его выполнение не влияет на получение зачёта по домашней работе.

**Что нужно сделать**

Используя конфигурации, выполненные в домашнем задании из предыдущего занятия, добавить к Production like сети Autoscaling group из трёх EC2-инстансов с  автоматической установкой веб-сервера в private домен.

1. Создать бакет S3 и разместить в нём файл с картинкой:

 - Создать бакет в S3 с произвольным именем (например, _имя_студента_дата_).
 - Положить в бакет файл с картинкой.
 - Сделать доступным из интернета.
2. Сделать Launch configurations с использованием bootstrap-скрипта с созданием веб-страницы, на которой будет ссылка на картинку в S3. 
3. Загрузить три ЕС2-инстанса и настроить LB с помощью Autoscaling Group.

Resource Terraform:

- [S3 bucket](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket)
- [Launch Template](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch_template).
- [Autoscaling group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group).
- [Launch configuration](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch_configuration).

Пример bootstrap-скрипта:

```
#!/bin/bash
yum install httpd -y
service httpd start
chkconfig httpd on
cd /var/www/html
echo "<html><h1>My cool web-server</h1></html>" > index.html
```
### Правила приёма работы

Домашняя работа оформляется в своём Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
