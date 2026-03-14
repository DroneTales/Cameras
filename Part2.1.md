# Китайские видеокамеры в Apple Home. Часть 2.1

В этой статье описано как добавить поддержку звука для вашей камеры в Apple Home.

Первая часть [видео](https://youtu.be/Dx2-p7UPayY) и [описания](./Part1.md).  
Вторая часть [видео](https://youtu.be/r_2EdMq5mtc) и [описания](./Part2.md).  

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
