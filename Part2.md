# Китайские видеокамеры в Apple Home. Часть 2

Это инструкция к [этому](https://youtu.be/r_2EdMq5mtc) видео.

Первая часть (видео)[https://youtu.be/Dx2-p7UPayY] и (описания)[./Part1.md]

## Определение MAC адреса видеокамеры

Подключитесь к вашей Raspberry Pi по SSH:

`ssh 192.168.1.13 -I homebridge`

Введите `homebridge` в качестве пароля.

После подключения введите следующую команду для получения списка всех назначенных динамических IP адресов:

`cat /var/lib/dhcp/dhcpd.leases`

Проверьте последний назначенный IP адрес. Скорее всего это и будет адрес видеокамеры. Запомните или скопируйте значение **hardware address****.

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
