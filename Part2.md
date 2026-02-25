# China cameras in Apple Home. Part 2

This file contains instructtions for [this](_____________) video.

## Find camera's MAC address

Connect to RPi via SSH

`ssh 192.168.1.13 -I homebridge`

Use `homebridge` as password.

Once connect use the following command to list all assigned dinamic IP addresses

`cat /var/lib/dhcp/dhcpd.leases`

Check out latest assigned IP. It should be our camera's IP address. Remember or copy the hardware address of the camera,

## Change DHCP configuration

Now add new record to the ISC DHCP server configuration so the camera gets static IP.

Open the configuration file by entering the following command

`sudo nano /etc/dhcp/dhcpd.conf`

And add the following line right before the `subnet 192.168.1.0`

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

Test configuration

`sudo dhcpd -t`

If no errors then restart the DHCP server

`sudo systemctl restart isc-dhcp-server`

Reboot the camera and make sure it is available on the specified IP address. You can use ping

`ping 192.168.1.100`

## Find RTSP stream

Use [this site](https://www.ispyconnect.com/camera/china) to find possible RTSP URL for your camera.

## Camera UI config

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
