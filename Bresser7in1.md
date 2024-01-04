# Weather Station with Home Assistant
## Hardware:
- Bresser 7in1 Weather Center ClimateScout (should working with any radio frequency weather station)
- USB TV Tuner (or any RTL-SDR receiver with RTL2832U)
![Bresser 7in1](https://github.com/tklakla/home_assistant/assets/54936857/43a8070d-1df8-45c6-9953-a279224ba58e)
![Elistooop TV Tuner](https://github.com/tklakla/home_assistant/assets/54936857/81a68740-d3db-4836-aea4-a04afbe04316)
## Software stack
- Linux Debian 11
- Docker Compose
- RTL 433 To MQTT (https://hub.docker.com/r/bademux/rtl_433tomqtt) or RTL 433 by hertzg (https://hub.docker.com/r/hertzg/rtl_433)
- Mosquitto (https://hub.docker.com/_/eclipse-mosquitto)
- Influx DB (https://hub.docker.com/_/influxdb)

1. Insert USB dongle into the host, check if is visible:
```
lsusb
```
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

2. Update USB rules in **/etc/udev/rules.d** creating **99-rtl_sdr.rules** file:
```
# RTL SDR`
SUBSYSTEM=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="2838", MODE="0666", GROUP="plugdev", SYMLINK+="rtl_sdr"
```
3. Now you can apply rule:
```
udevadm control --reload-rules && udevadm trigger
```
4. Update your docker-compose.yml file:
```
  rtl-433tomqtt:
    container_name: rtl-433tomqtt
    image: bademux/rtl_433tomqtt:latest
    restart: unless-stopped
    volumes:
      - ./rtl_433.conf:/etc/rtl_433/rtl_433.conf:ro
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
    devices:
      - /dev/bus/usb:/dev/bus/usb
```
5. Edit rtl_433.conf file:
```
frequency     868.30M
report_meta stats:2:60
report_meta time:tz
output json
output mqtt://192.168.8.9:1883,retain=1,user=MQTT_USER,pass=MQTT_PASSWORD,devices=rtl433/devices[/type][/model][/subtype][/channel][/id]
output influx://192.168.8.9:8086/api/v2/write?org=INFLUX_ORG&bucket=INFLUX_BUCKET,token=INFLUX_TOKEN
convert si
protocol 173 # Bresser Weather Center 7-in-1
```
6. Run new docker service:
```
docker compose up -d
```
> Note:
>>  During testing, changing the USB port helps
>> 
7. Check that everything is working:
- MQTT Explorer:

   ![MQTT Explorer](https://github.com/tklakla/home_assistant/assets/54936857/b91c3af6-c242-4b1b-b447-11aff211d90f)
- InfluxDB:

   ![Influx DB](https://github.com/tklakla/home_assistant/assets/54936857/6f50d8d6-beab-405f-8df5-36314467a748)
- Grafana:

  ![Grafana](https://github.com/tklakla/home_assistant/assets/54936857/14481a84-c5fc-4cb5-94fd-65a3e7943911)

8. And finally, add Home Assistant sensors to configuration.yaml:
```
mqtt:
  sensor:
    - name: "Outside Temperature"
      state_topic: "rtl433/devices/Bresser-7in1/43971/temperature_C"
      state_class: measurement
      device_class: temperature
      unit_of_measurement: '°C'
    - name: "Outside Humidity"
      state_topic: "rtl433/devices/Bresser-7in1/43971/humidity"
      state_class: measurement
      device_class: humidity
      unit_of_measurement: '%'
```
9. That's all Folks!:

![image](https://github.com/tklakla/home_assistant/assets/54936857/786f60f0-3446-4723-a2be-a4fa22c97c89)
