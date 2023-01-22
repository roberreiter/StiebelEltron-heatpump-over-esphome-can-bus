This is a config for an esphome device, together with an MCP2515 can bus component.
Descriptions how to build up the module and further discussions can be found in my thread in the home assistant forum:
https://community.home-assistant.io/t/configured-my-esphome-with-mcp2515-can-bus-for-stiebel-eltron-heating-pump/366053/16

Eventually you have to change the CAN-id of the esphome device, if you get no response from the bus.
Some users reported that the CAN-id 680 didn't work for them.
Also some users reported from different baud-rates.
Change them in the section, where you configure the canbus + change the gloabal variable internalResponse_id respectively.

For the address of the heat pump, change the variables PumpCANread_id and PumpCANwrite_id.
For the address of the FEK, change the variable FekCANread_id
