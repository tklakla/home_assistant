# Weather Station with Home Assistant
## Hardware:
- Bresser 7in1 Weather Center ClimateScout
- USB TV Tuner (SDR, RTL 2832U)
![Bresser 7in1](https://github.com/tklakla/home_assistant/assets/54936857/43a8070d-1df8-45c6-9953-a279224ba58e)
![Elistooop TV Tuner](https://github.com/tklakla/home_assistant/assets/54936857/81a68740-d3db-4836-aea4-a04afbe04316)
## Software stack
- Linux Debian 11
- Docker Compose
- RTL 433 To MQTT (https://hub.docker.com/r/bademux/rtl_433tomqtt) or RTL 433 by hertzg (https://hub.docker.com/r/hertzg/rtl_433)
- Mosquitto (https://hub.docker.com/_/eclipse-mosquitto)
- Influx DB (https://hub.docker.com/_/influxdb)

1. Insert USB dongle into the host, check if is visible:
   
`lsusb`
> Bus 005 Device 010: ID 0bda:2838 Realtek Semiconductor Corp. RTL2838 DVB-T
> 
> Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
> 
> Bus 004 Device 003: ID 05ac:8240 Apple, Inc. Built-in IR Receiver
> 
> Bus 004 Device 004: ID 05ac:8205 Apple, Inc. Bluetooth HCI
> 
> Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
> 
> Bus 003 Device 002: ID 413c:2107 Dell Computer Corp. KB212-B Quiet Key Keyboard
> 
> Bus 003 Device 003: ID 413c:301a Dell Computer Corp. Dell MS116 Optical Mouse
> 
> Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
> 
> Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
> 
> Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub

  It means idVendor=Obda, idProduct=2838

2. Update USB rules in **/etc/udev/rules.d** creating **99-rtl_sdr.rules** file
