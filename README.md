This is a config for an esphome device, together with an MCP2515 can bus component.
Descriptions how to build up the module and further discussions can be found in my thread in the home assistant forum:
https://community.home-assistant.io/t/configured-my-esphome-with-mcp2515-can-bus-for-stiebel-eltron-heating-pump/366053/16

Eventually you have to change the CAN-id of the esphome device, if you get no response from the bus.
Some users reported that the CAN-id 680 didn't work for them.
Also some users reported from different baud-rates.
Change them in the section, where you configure the canbus + change the gloabal variable internalResponse_id respectively.
You will further have to change all values that correspond to 0x680 and the can-id 680


For the address of the heat pump, change the variables PumpCANread_id and PumpCANwrite_id.
For the address of the FEK, change the variable FekCANread_id

#
To change the comfort-temperature and eco-temperature of your heat-pump via home-assistant, add the following sensors to your homeassistant config.yaml:

input_text:

  ww_komfort_temp:
    name: Warmwassertemperatur Komfort 1/10°C
    min: 3
    max: 3
    pattern: "[0-9]*"

  ww_eco_temp:
    name: Warmwassertemperatur Eco 1/10°C
    min: 3
    max: 3
    pattern: "[0-9]*"


#
Insert a value in 1/10 degrees Celsius.
e.g. to set the comfort-temperature of your heat-pump to 55°C, set the value to 550
Changing a value in the input_text will directly generate a change the value on your heat-pump.
You can verify it on your heat-pump's display.
I personally change my comfort-temperature when there is enough energy-production from my photovoltaic module and leave the eco-temperature constantly low.


#
#
To send individual CAN-messages from Homeassistant there is an implementation in my code, where you have to add some sensors and a script in Home-Assistant.
This is more likely for experimental use, but it can help to find useful messages from your heatpump (as they sometimes vary from one device to the other).
Be aware, that you can also change values on your heatpump with this function!
Howto:


#
Add the following sensors to your homeassistant config.yaml, to create input-textfields:


input_text:

  h_addr:
    name: h_addr
    initial: "3100"
    min: 4
    max: 4
    pattern: "[a-fA-F0-9]*"
  h_idx:
    name: h_idx
    initial: "0000"
    min: 4
    max: 4
    pattern: "[a-fA-F0-9]*"
  h_val:
    name: h_val
    initial: "0000"
    min: 4
    max: 4
    pattern: "[a-fA-F0-9]*"

#
Add the following script to you homeassistant scripts.yaml-file.
The script reads the values from the input-text, converts the values into integer-values and forwards them to the ESPhome sensor.
This is to prepare the can-message for forwarding it to the CAN-bus. The script itself does not forward a value to the bus.
It is just generating the message in integer. The sending-part is initiated using the buttons on the esphome-device.


sequence:
  - service: esphome.<your_esphome_device_name>
    data:
      idx: "{{ states(\"input_text.h_idx\")|int(base=16) }}"
      addr: "{{ states(\"input_text.h_addr\")|int(base=16) }}"
      val: "{{ states(\"input_text.h_val\")|int(base=16) }}"
mode: queued
icon: mdi:play
max: 5
alias: send CAN-message to ESP-home

#
Create a button for the script in your Lovelace interface

#
To send an individual CAN-message, insert the hex-values for address in h_addr.
e.g. 3100 to receive a value or 3000 to change a value on your heat-pump with address 0x180. Change the address to your individual need.

Insert the Elster-index of interest in h_idx. e.g. (000c for outside-temperature)

Insert a value in h_val (if you want to change a value) or leave it as 0000 if you just want to receive a current value.
Press the button "send CAN-message to ESP-home

Press the button "Befehl anzeigen / Sensorupdate" that is provided by the esphome-device, to show up the CAN-message in the log.
Press the button "CAN-Befehl absetzen" that is provided by the esphome-device, to send the CAN-message to the bus.
If you did everything correctly, the esphome-log will show an answer to your CAN-request.

Other noteworthy addresses and their hex-values for the CAN-message
#
#CAN ID 180: read - 3100, write - 3000
#CAN ID 301: read - 0c01, FEK-device (no active can request, only listening)
#
#other addresses
#   180 read: 3100  write: 3000
#  	301	read: 6101  write: 6001
#	480	read: 9100  write: 9000    WMPme Wärmepumpenmanager
#	601	read: C101  write: C001
#	680	confirmation: D200
#
