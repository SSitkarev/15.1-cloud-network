# Домашнее задание к занятию «Организация сети» - Сергей Ситкарёв

## Задание 1. Yandex Cloud

### 1. Создать пустую VPC. Выбрать зону.

```
resource "yandex_vpc_network" "VPC" {
  name = "VPC"
}
```

![Задание1](https://github.com/SSitkarev/15.1-cloud-network/blob/main/img/1.jpg)

### 2. Публичная подсеть.

Создать в VPC subnet с названием public, сетью 192.168.10.0/24.

```
resource "yandex_vpc_subnet" "subnet-public" {
  name           = "public"
  zone           = var.default_zone
  network_id     = yandex_vpc_network.VPC.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}
```

![Задание2](https://github.com/SSitkarev/15.1-cloud-network/blob/main/img/2.jpg)

Создать в этой подсети NAT-инстанс, присвоив ему адрес 192.168.10.254. В качестве image_id использовать fd80mrhj8fl2oe87o4e1.

```
resource "yandex_vpc_subnet" "subnet-private" {
  name           = "private"
  zone           = var.default_zone
  network_id     = yandex_vpc_network.VPC.id
  route_table_id = yandex_vpc_route_table.nat-route-table.id
  v4_cidr_blocks = ["192.168.20.0/24"]
}

resource "yandex_vpc_route_table" "nat-route-table" {
  network_id = yandex_vpc_network.VPC.id

  static_route {
    destination_prefix = "0.0.0.0/0"
    next_hop_address   = "192.168.10.254"
  }
}
```

```
resource "yandex_compute_instance" "nat-instance" {
  name     = var.nat-instance-name
  hostname = "${var.nat-instance-name}.${var.domain}"
  zone     = var.default_zone

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id    = var.nat-instance-image-id
      name        = "root-${var.nat-instance-name}"
      type        = "network-nvme"
      size        = "50"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-public.id
    ip_address = var.nat-instance-ip
    nat       = true
  }

  metadata = {
    ssh-keys = "centos:${var.public_key}"
  }
}
```

![Задание2](https://github.com/SSitkarev/15.1-cloud-network/blob/main/img/3.jpg)

Создать в этой публичной подсети виртуалку с публичным IP, подключиться к ней и убедиться, что есть доступ к интернету.

```
resource "yandex_compute_instance" "public-vm" {
  name     = var.public-vm-name
  hostname = "${var.public-vm-name}.${var.domain}"
  zone     = var.default_zone

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id    = var.centos-7-base
      name        = "root-${var.public-vm-name}"
      type        = "network-nvme"
      size        = "50"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-public.id
    nat       = true
  }

  metadata = {
    ssh-keys = "centos:${var.public_key}"
  }
}
```

![Задание2](https://github.com/SSitkarev/15.1-cloud-network/blob/main/img/4.jpg)

### 3. Приватная подсеть.

Создать в VPC subnet с названием private, сетью 192.168.20.0/24.

```
resource "yandex_vpc_subnet" "subnet-private" {
  name           = "private"
  zone           = var.default_zone
  network_id     = yandex_vpc_network.VPC.id
  route_table_id = yandex_vpc_route_table.nat-route-table.id
  v4_cidr_blocks = ["192.168.20.0/24"]
}
```

Создать route table. Добавить статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс.

```
resource "yandex_vpc_route_table" "nat-route-table" {
  network_id = yandex_vpc_network.VPC.id

  static_route {
    destination_prefix = "0.0.0.0/0"
    next_hop_address   = "192.168.10.254"
  }
}
```

Создать в этой приватной подсети виртуалку с внутренним IP

```
resource "yandex_compute_instance" "private-vm" {
  name     = var.private-vm-name
  hostname = "${var.private-vm-name}.${var.domain}"
  zone     = var.a-zone

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id    = var.centos-7-base
      name        = "root-${var.private-vm-name}"
      type        = "network-nvme"
      size        = "50"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-private.id
    nat       = false
  }

  metadata = {
    ssh-keys = "centos:${file("id_ed25519.pub")}"
  }
}
```

![Задание3](https://github.com/SSitkarev/15.1-cloud-network/blob/main/img/5.jpg)

Подключиться к ней через виртуалку, созданную ранее, и убедиться, что есть доступ к интернету.

![Задание3](https://github.com/SSitkarev/15.1-cloud-network/blob/main/img/6.jpg)