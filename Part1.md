# Китайские видеокамеры в Apple Home. Часть 1

Здесь вы найдете инструкции [этому](https://youtu.be/Dx2-p7UPayY) видео.

Вторая часть [видео](https://youtu.be/r_2EdMq5mtc) и [описания](./Part2.md).  

## Установка обновлений для Raspbian

Откройте консоль или подключитель к Raspberry Pi по SSH и введите следующие команды для установки последних обновлений Raspbian OS:

`sudo apt-get update`  
`sudo apt-get upgrade`  

## Установка и настройка isc-dhcp-server

Введите следующую команду для установки **ISC DHCP server**:

`sudo apt-get install isc-dhcp-server`

Проверьте сетевой интерфейс, используемый вышей Raspberry Pi:

`sudo ip a`

Обычно это **eth0**.

Откройте файл настройки интерфейсов isc-dhcp-server:

`sudo nano /etc/default/isc-dhcp-server`

Измените **INTERFACESv4** строку, добавив ваш сетевой интерфейс. В случае, если это **eth0** строка должны выглядеть следующим образом:

`INTERFACESv4="eth0"`

Сохраните изменения, нажав следущие клавиши **Control+X**, **Y**, **Return**.

Откройте конфигурационный файл введя следующую команду:

`sudo nano /etc/dhcp/dhcpd.conf`

Измените файл как показано ниже.

```
option domain-name "homebridge.local";

default-lease-time 600;
max-lease-time 7200;

host mikebook {
  hardware ethernet 7E:16:2D:5D:66:3A;
  fixed-address 192.168.1.20;
  option subnet-mask 255.255.255.0;
  option routers 192.168.1.1;
  option domain-name-servers 8.8.8.8;
  default-lease-time 604800;
  max-lease-time 2592000;
}

subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.200 192.168.1.250;
  option subnet-mask 255.255.255.0;
  option routers 192.168.1.1;
  option domain-name-servers 8.8.8.8;
}
```

Замените **mikebook** на сетевое имя вашего компьютера. Замените MAC адрсе из пример (**7E:16:2D:5D:66:3A**) на MAC адрес вашего ноутбука/компьютера.

Сохраните изменения нажав **Control+X**, **Y**, **Return**.

Проверьте правильность конфигурации:

`sudo dhcpd -t /etc/dhcp/dhcpd.conf`

Перезапустите ISC DHCP server:

`sudo systemctl restart isc-dhcp-server`

## Установка HomeBridge

Добавьте GPG ключ от репозитория Homebridge:

`curl -sSfL https://repo.homebridge.io/KEY.gpg | sudo gpg --dearmor | sudo tee /usr/share/keyrings/homebridge.gpg  > /dev/null`

Добавьте репозиторий Homebridge в системный список репозиториев:

`echo "deb [signed-by=/usr/share/keyrings/homebridge.gpg] https://repo.homebridge.io stable main" | sudo tee /etc/apt/sources.list.d/homebridge.list > /dev/null`

Обновите репозитории:

`sudo apt-get update`

Установите Homebridge:

`sudo apt-get install homebridge`

Готово.
