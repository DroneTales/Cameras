Это руководство к серии видео про добавление дешевых китайских видеокамер в Apple Home с использованием HomeBridge.

## Содержание

 1. [Часть 1](#китайские-видеокамеры-в-apple-home.-часть-1)  
    - [Установка обновлений для Raspbian](#установка-обновлений-дляraspbian)
    - [Установка и настройка isc-dhcp-server](#установка-и-настройка-isc-dhcp-server)
    - [Установка HomeBridge](#установка-homebridge)

### Ссылки на видео

 - [Первая часть](https://youtu.be/Dx2-p7UPayY)  
 - [Вторая часть](https://youtu.be/r_2EdMq5mtc)  
 - [Третья часть](https://youtube.com/shorts/5GIIGDH4DWg)


# Китайские видеокамеры в Apple Home. Часть 1

В этой части мы подготовим Raspberry Pi (или другое ваше устрйоство, которое будет выступать в качестве сервера HomeBridge: компьютер, ноутбук) для установки на него HomeBridge.

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

`sudo apt-get install homebridge`. 

# Китайские видеокамеры в Apple Home. Часть 2

В этой части мы подключим камеры к HomeBridge и добавим их в Apple Home.

## Определение MAC адреса видеокамеры

Подключитесь к вашей Raspberry Pi по SSH:

`ssh 192.168.1.13 -I homebridge`

Введите `homebridge` в качестве пароля.

После подключения введите следующую команду для получения списка всех назначенных динамических IP адресов:

`cat /var/lib/dhcp/dhcpd.leases`

Проверьте последний назначенный IP адрес. Скорее всего это и будет адрес видеокамеры. Запомните или скопируйте значение **hardware address**.

## Изменяем настройки DHCP сервера

Теперь добавьте новый запись в конфигурационный файл ISC DHCP что бы для камеры всегда выделялся статический IP. Для этого откройте конфигурационный файл введя следующую команду:

`sudo nano /etc/dhcp/dhcpd.conf`

Добавьте следующие строки сразу **перед** `subnet 192.168.1.0`:

```
host ipecam {  
    hardware ethernet 30:4A:26:72:1E:3E;  
    fixed-address 192.168.1.100;  
    option subnet-mask 255.255.255.0;  
    option routers 192.168.1.1;  
    option domain-name-servers 8.8.8.8;  
    default-lease-time 604800;  
    max-lease-time 2592000;  
}
```

Проверьте конфигурацию следующей командой:

`sudo dhcpd -t`

Если ошибок нет, перезапустите DHCP сервер:

`sudo systemctl restart isc-dhcp-server`

Перезагрузите видеокамеру и проверьте, что она получила выделенный IP. Используйте команду ping:

`ping 192.168.1.100`

## Ищем RTSP поток камеры

Используйте [этот сайт](https://www.ispyconnect.com/camera/china) для поиска возможного RTSP URL для вашей камеры.

## Конфигурация Camera UI

Замените **source**, **subSource** и **stillImageSource** значениями RTSP URL для вашей камеры.

```
{
    "platform": "CameraUI",
    "name": "CameraUI",
    "port": 8081,
    "atHomeSwitch": false,
    "logLevel": "info",
    "mqtt": {
            "active": false,
            "tls": false,
            "port": 1883
        },
    "http": {
            "active": false,
            "port": 7272,
            "localhttp": false
        },
        "smtp": {
            "active": false,
            "port": 2727,
            "space_replace": "+"
        },
        "ftp": {
            "active": false,
            "useFile": false,
            "port": 5050
        },
        "ssl": {
            "active": false
        },
    "cameras": [
        {
            "name": "IPCam",
            "manufacturer": "GK7102",
            "model": "YCC365",
            "serialNumber": "6-7-2-102",
            "unbridge": true,
            "prebuffering": true,
            "prebufferLength": 4,
            "hsv": true,
            "videoConfig": {
                "source": "-i rtsp://admin:@192.168.1.100:8001/0/av0",
                "subSource": "-i rtsp://admin:@192.168.1.100:8001/0/av1",
                "stillImageSource": "-i rtsp://admin:@192.168.1.100:8001/0/av1",
                "vcodec": "copy",
                "acodec": "aac",
                "audio": false
            }
        }
    ],
    "options": {
        "videoProcessor": "ffmpeg"
    }
}
```

# Китайские видеокамеры в Apple Home. Часть 3.

В этой статье описано как добавить поддержку звука для вашей камеры в Apple Home.

## Установка aac кодека для HomeBridge

Подключитесь к Raspberry Pi по SSH и выполните следующую команду:

`sudo curl -Lf# https://github.com/homebridge/ffmpeg-for-homebridge/releases/latest/download/ffmpeg-alpine-$(uname -m).tar.gz | sudo tar xzf - -C / --no-same-owner`

## Изменение конфигурации Camera UI

Откройте WEB интерфейс HomeBridge, перейдите в раздел **Редактор JSON**. Найдите вашу камеру в разделе **cameras**. В раздеое **videoConfig** для вашей камеры установите параметр **audio** в **true**, параметр **acodec** в **libfdk_aac2**. Теперь эта часть конфигурации должна вышлядеть как в примере ниже

```
...
acodec": "libfdk_aac2",
"audio": true
...
```

Перезапустите HomeBridge. Готово, теперь у вас камеры могу писать звук.
