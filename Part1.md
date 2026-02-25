
## Install Raspbian Updates

Execute the following commands to install latest Raspbian OS updates.

`sudo apt-get update`  
`sudo apt-get upgrade`  

## Install and configure isc-dhcp-server

Execute the following command to install **ISC DHCP server**.

`sudo apt-get install isc-dhcp-server`

Check the network interface used by your Raspberry Pi

`sudo ip a`

Usually it is **eth0**.

Open the isc-dhcp-server interface file

`sudo nano /etc/default/isc-dhcp-server`

Modify the **INTERFACESv4** line by adding your network interface. In case if it is **eth0** the line should look like this:

`INTERFACESv4="eth0"`

Save the file by typing **Control+X**, **Y**, **Return**.

Open configuration file.

`sudo nano /etc/dhcp/dhcpd.conf`

Modify the file as below.

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

Raplcase **mikebook** with your notebook or PC name. Replace the MAC address (**7E:16:2D:5D:66:3A**) with your notebook/PC MAC.

Save the file by typing **Control+X**, **Y**, **Return**.

Test configuration

`sudo dhcpd -t /etc/dhcp/dhcpd.conf`

Restart the ISC DHCP server

`sudo systemctl restart isc-dhcp-server`

## Install HomeBridge

Add the Homebridge Repository GPG key:

`curl -sSfL https://repo.homebridge.io/KEY.gpg | sudo gpg --dearmor | sudo tee /usr/share/keyrings/homebridge.gpg  > /dev/null`

Add the Homebridge Repository to the system sources:

`echo "deb [signed-by=/usr/share/keyrings/homebridge.gpg] https://repo.homebridge.io stable main" | sudo tee /etc/apt/sources.list.d/homebridge.list > /dev/null`

Update repositories:

`sudo apt-get update`

Install Homebridge:

`sudo apt-get install homebridge`

Done.
