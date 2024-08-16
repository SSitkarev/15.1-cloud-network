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

Создать в этой подсети NAT-инстанс, присвоив ему адрес 192.168.10.254. В качестве image_id использовать fd80mrhj8fl2oe87o4e1.

Создать в этой публичной подсети виртуалку с публичным IP, подключиться к ней и убедиться, что есть доступ к интернету.

### 3. Приватная подсеть.

Создать в VPC subnet с названием private, сетью 192.168.20.0/24.

Создать route table. Добавить статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс.

Создать в этой приватной подсети виртуалку с внутренним IP, подключиться к ней через виртуалку, созданную ранее, и убедиться, что есть доступ к интернету.