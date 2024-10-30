## Домашнее задание к занятию «Организация сети»
### Задание 1. Yandex Cloud

####  Создать пустую VPC. Выбрать зону.
  
####  Публичная подсеть.
  ```
  # Создание облачной сети

resource "yandex_vpc_network" "vpc" {
  name = local.network_name
}

  ```
  Создать в VPC subnet с названием public, сетью 192.168.10.0/24.
  ```
resource "yandex_vpc_subnet" "public-subnet" {
  name           = local.subnet_name1
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.vpc.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}
  ```
  Создать в этой подсети NAT-инстанс, присвоив ему адрес 192.168.10.254.
  ```
resource "yandex_compute_instance" "nat-instance" {
  name        = local.vm_nat_name
  platform_id = "standard-v3"
  zone        = "ru-central1-a"

  resources {
    core_fraction = 20
    cores         = 2
    memory        = 2
  }

  boot_disk {
    disk_id = yandex_compute_disk.boot-disk-nat.id
  }

  network_interface {
    subnet_id          = yandex_vpc_subnet.public-subnet.id
    nat                = true
    ip_address         = "192.168.10.254"
  }

  metadata = {
    user-data = "#cloud-config\nusers:\n  - name: ${var.vm_user_nat}\n    groups: sudo\n    shell: /bin/bash\n    sudo: 'ALL=(ALL) NOPASSWD:ALL'\n    ssh_authorized_keys:\n      - ${file("${var.ssh_key_path}")}"
  }
}
  ```
В качестве image_id использовать fd80mrhj8fl2oe87o4e1.
  ```
resource "yandex_compute_disk" "boot-disk-nat" {
  name     = "boot-disk-nat"
  type     = "network-hdd"
  zone     = "ru-central1-a"
  size     = "10"
  image_id = "fd80mrhj8fl2oe87o4e1"
}
```
  
  Создать в этой публичной подсети виртуалку с публичным IP, подключиться к ней и убедиться, что есть доступ к интернету.
  ```
resource "yandex_compute_instance" "test-vm" {
  name        = local.vm_test_name
  platform_id = "standard-v3"
  zone        = "ru-central1-a"

  resources {
    core_fraction = 20
    cores         = 2
    memory        = 2
  }

  boot_disk {
    disk_id = yandex_compute_disk.boot-disk-ubuntu.id
  }

  network_interface {
    subnet_id          = yandex_vpc_subnet.public-subnet.id
    nat = true
  }

  metadata = {
    user-data = "#cloud-config\nusers:\n  - name: ${var.vm_user}\n    groups: sudo\n    shell: /bin/bash\n    sudo: 'ALL=(ALL) NOPASSWD:ALL'\n    ssh_authorized_keys:\n      - ${file("${var.ssh_key_path}")}"
  }
}
  ```
![изображение](https://github.com/user-attachments/assets/c28bd264-7b7e-44f3-9691-462bc4a401dc)

![изображение](https://github.com/user-attachments/assets/7c0c7d8c-8512-47e9-bc59-53003836b8b3)


    Приватная подсеть.

    Создать в VPC subnet с названием private, сетью 192.168.20.0/24.
    Создать route table. Добавить статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс.
    Создать в этой приватной подсети виртуалку с внутренним IP, подключиться к ней через виртуалку, созданную ранее, и убедиться, что есть доступ к интернету.

