Honeywell Resideo R200C2-A Mod for ESPHome
=========================
MQTT VERSION HERE https://github.com/Pluimvee/Resideo

Custom component for ESPHome to snif traffic between the Resideo firmware and the
- cht8305 chip (I2C) measuring humidity and temperature
- cm1106 chip (RS232) measuring CO2 levels

![image](https://github.com/Pluimvee/Resideo/assets/124380379/37667938-26ab-41bc-9024-0448e24f58d0)

Usage
-----
I noticed differences between what the Resideo displays on screen and the values received from the cht8305 sensor, converted according the the specs. The sensor should is calibrated during manufacturing so the received values should be correct. However the casing may effect the values and this may have been adjusted by Resideo (Honewell) in the firmware. You can use the offset values using the standard filters of esphome to match the values with what's displayed on screen.

The `cm1106_sniffer` component is a proper ESPHome `uart::UARTDevice`. You must declare a `uart:` block and configure the RX pin that is connected to the CM1106 data line.

```yaml
external_components:
  - source: github://Pluimvee/esphome-resideo
    components: [cht8305_sniffer, cm1106_sniffer]

# ESP8266: UART0 RX is GPIO3. On ESP32 any RX-capable pin can be used.
uart:
  baud_rate: 9600
  rx_pin: GPIO3

sensor:
  - platform: cht8305_sniffer
    temperature:
      name: "Temperature"
      filters:
        - calibrate_linear: # adjust temperature if needed
          - 0.00 -> 0.00
          - 26.5 -> 20.0
    humidity:
      name: "Humidity"
      filters:
        - offset: 2.1  # Adjust humidity offset if needed

  - platform: cm1106_sniffer
    # update_interval: 10s  (default 5s)
    name: "CO2 Level"
```

If your board has multiple UARTs, pass the `uart_id` explicitly:

```yaml
uart:
  id: co2_uart
  baud_rate: 9600
  rx_pin: GPIO16

sensor:
  - platform: cm1106_sniffer
    uart_id: co2_uart
    name: "CO2 Level"
```

NOTE
-----
Be sure to disable the use of UART0 by the logger by setting `baud_rate: 0`. This releases the hardware serial port so it can be used by the `uart:` component.

```yaml
logger:
  level: INFO
  baud_rate: 0  # release UART0 for cm1106_sniffer
```

# Hardware
- 4x Honeywell voor €84 (inc. shipment) https://www.ibood.com/nl/s-nl/o/2x-honeywell-resideo-co2-monitor-detector/987304
- 4x ESP-12F voor €6 (inc. shipment) https://nl.aliexpress.com/item/4001157391459.html

**€23 per unit**

# Wiring
The wiring of the ESP8266 ESP12F module to the Honeywell Resideo

![image](https://github.com/Pluimvee/Resideo/assets/124380379/716bbd6b-b180-443f-b0d4-bdce23c670cb)

# Research
Sniffing the communication of the CHT8305 Humidity/Temperature sensor I used the below code. This code is using interrupts for more stability

https://github.com/Pluimvee/I2C-sniffer/blob/main/I2C-sniffer.ino

The SDA/SCL pins can also be found on the display-board-connector, and on the backside of the display-board. I tried to solder some pins on the backside of the display-board however the test-pads are to week to hold any pins. Therefore I soldered some pins into the pads next to the battery connector.

Sniffing the communication of the CM1106 sensor I used a TTL-2-USB bridge
https://nl.aliexpress.com/item/1005002007754292.html

Using this module I found out that the pins TX and RX on the CM1106 module have the same data streams as found on the D and C pins behind the display module. 
- The C has the same data as found on the RX pin of the CO2 module -> 11 01 01 ED. Therefore I think C stands for Command and the instruction send by the controller to the CO2 module is found there. 
- The D has the same data as found on the TX pin of the CO2 module -> 16 05 01 DF1-DF4 CS. I asume D stands for Data and there we can find the response of the CO2 module send to the controller.

Therefore we only need to connect the D(ata) pin to the RX of your ESP, and do not need to solder anything on the CM1106 module itself.

![image](https://github.com/Pluimvee/Resideo/assets/124380379/266c5ccd-abe3-4957-84f4-51ea9856ff9a)

My graditude to the research and work of the following tweakers:
- Bartvb (https://gathering.tweakers.net/forum/list_message/78228570#78228570)
- Soepstengel (https://gathering.tweakers.net/forum/list_message/77882454#77882454)
- Skix_Aces (https://gathering.tweakers.net/forum/list_message/78427188#78427188)
- ThinkPad (https://gathering.tweakers.net/forum/list_message/77642990#77642990)




