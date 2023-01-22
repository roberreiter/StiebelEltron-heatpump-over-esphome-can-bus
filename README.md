# StiebelEltron-heatpump-over-esphome-can-bus
#ESPhome-configuration to read and write values on a Stiebel Eltron heat pump

esphome:
  name: esp-sensor-node-hzg
  platform: ESP32
  board: esp32dev

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: "<password>"


# Service to pull individual can message from home assistant
  services:
    - service: pull_canmsg
      variables:
        idx: int
        addr: int
        val: int
      then:
        - lambda: |-
            int getA = static_cast<int>(addr); //Adresse in Byte 0 u. 1 schreiben
            id(sh_state)[0]=getA>>8;
            id(sh_state)[1]=getA-((getA>>8)<<8);

            getA = static_cast<int>(idx); //Elster-Index übernehmen
            //Wenn Elster-Index <= 0xff => an Byte-Stelle 2 schreiben
            if( (getA>>8) == 0x00) {
              id(sh_state)[2]=getA-((getA>>8)<<8);

              getA = static_cast<int>(val); //Datenwert übernehmen und an Stelle 3 u. 4 schreiben, 5 u. 6 ist 0x00
              id(sh_state)[3]=getA>>8;
              id(sh_state)[4]=getA-((getA>>8)<<8);
              id(sh_state)[5]=0x00;
              id(sh_state)[6]=0x00;
            }
            else {
              //Wenn Elster-Index > 0xff kommt 0xfa an Stelle 2, der Index steht dann an Stelle 3 u. 4
              id(sh_state)[2]=0xfa;
              id(sh_state)[3]=getA>>8;
              id(sh_state)[4]=getA-((getA>>8)<<8);

              getA = static_cast<int>(val); //der Datenwert steht dann an Stelle 5 u. 6
              id(sh_state)[5]=getA>>8;
              id(sh_state)[6]=getA-((getA>>8)<<8);
            }




ota:
  password: "<password>"

wifi:
  ssid: "<ssid>"
  password: "<password>"

  use_address: <ip_address>


  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "<fallback_ssid>"
    password: "<password>"

captive_portal:

globals:

#################################################################
#Stiebel Eltron WPF-07 Cool 2018
#WPM3i software version 391-08
#FEK software version 416 - 02
#################################################################
#CAN ID 180: read - 3100, write - 3000
#CAN ID 301: read - 0c01, FEK-device (no active can request, only listening)
#
#other addresses
#   180 read: 3100  write: 3000
#  	301	read: 6101  write: 6001
#	480	read: 9100  write: 9000    WMPme Wärmepumpenmanager
#	601	read: C101  write: C001
#	680	confirmation: D200
#################################################################
#change this IDs if required
  
#CAN read address for heat pump manager  
  - id: PumpCANread_id
    type: int[2]
    initial_value: '{0x31, 0x00}'
    restore_value: no
#CAN write adress for heat pump manager
  - id: PumpCANwrite_id
    type: int[2]
    initial_value: '{0x30, 0x00}'
    restore_value: no

#CAN_ID of FEK
  - id: FekCANread_id
    type: int[2]
    initial_value: '{0x0c, 0x01}'
    restore_value: no
        
#CAN ID of esp-home device is set to 680 here, change this if you have to change the CAN-id
#Don't forget to also change the CAN-ID below
  - id: internalResponse_id
    type: int[2]
    initial_value: '{0xd2, 0x00}'
    restore_value: no
################################################################

#Array declaration to send CAN-Bus message from homeassistant
  - id: sh_state
    type: int[7]
    initial_value: '{0x00,0x00,0x00,0x00,0x00,0x00,0x00}'
    restore_value: no
#Array declaration to send CAN-Bus message from programcode
  - id: send_state
    type: int[7]
    initial_value: '{0x00,0x00,0x00,0x00,0x00,0x00,0x00}'
    restore_value: no

#declaration of sensor variables
  - id: el_aufnahmeleistung_ww_tag_wh_float
    type: float
    restore_value: no
  - id: el_aufnahmeleistung_ww_tag_wh_flag
    type: bool
    restore_value: no
  - id: el_aufnahmeleistung_ww_tag_kwh
    type: float
    restore_value: no
  - id: el_aufnahmeleistung_ww_tag_kwh_flag
    type: bool
    restore_value: no
  - id: el_aufnahmeleistung_heiz_tag_wh_float
    type: float
    restore_value: no
  - id: el_aufnahmeleistung_heiz_tag_wh_flag
    type: bool
    restore_value: no
  - id: el_aufnahmeleistung_heiz_tag_kwh
    type: float
    restore_value: no
  - id: el_aufnahmeleistung_heiz_tag_kwh_flag
    type: bool
    restore_value: no 
  - id: el_aufnahmeleistung_ww_total_kWh_float
    type: float
    restore_value: no
  - id: el_aufnahmeleistung_ww_total_kWh_flag
    type: bool
    restore_value: no
  - id: el_aufnahmeleistung_ww_total_mWh
    type: float
    restore_value: no
  - id: el_aufnahmeleistung_ww_total_mWh_flag
    type: bool
    restore_value: no
  - id: el_aufnahmeleistung_heiz_total_kWh_float
    type: float
    restore_value: no
  - id: el_aufnahmeleistung_heiz_total_kWh_flag
    type: bool
    restore_value: no
  - id: el_aufnahmeleistung_heiz_total_mWh
    type: float
    restore_value: no
    
  - id: volumenstrom_float
    type: float
    restore_value: no

  - id: el_aufnahmeleistung_heiz_total_mWh_flag
    type: bool
    restore_value: no

  - id: VD_starts_h
    type: int
    initial_value: '0'
    restore_value: no

  - id: VD_starts_t
    initial_value: '0'
    type: float
    restore_value: no

  - id: waermemertrag_ww_tag_wh_float
    type: float
    restore_value: no
  - id: waermemertrag_ww_tag_wh_flag
    type: bool
    restore_value: no
  - id: waermemertrag_ww_tag_kwh
    type: float
    restore_value: no
  - id: waermemertrag_ww_tag_kwh_flag
    type: bool
    
  - id: waermemertrag_electr_ww_tag_wh_float
    type: float
    restore_value: no
  - id: waermemertrag_electr_ww_tag_wh_flag
    type: bool
    restore_value: no
  - id: waermemertrag_electr_ww_tag_kwh
    type: float
    restore_value: no
  - id: waermemertrag_electr_ww_tag_kwh_flag
    type: bool    
    restore_value: no

  - id: waermemertrag_heiz_tag_wh_float
    type: float
    restore_value: no
  - id: waermemertrag_heiz_tag_wh_flag
    type: bool
    restore_value: no
  - id: waermemertrag_heiz_tag_kwh
    type: float
    restore_value: no
  - id: waermemertrag_heiz_tag_kwh_flag
    type: bool
    restore_value: no 

  - id: waermemertrag_electr_heiz_tag_wh_float
    type: float
    restore_value: no
  - id: waermemertrag_electr_heiz_tag_wh_flag
    type: bool
    restore_value: no
  - id: waermemertrag_electr_heiz_tag_kwh
    type: float
    restore_value: no
  - id: waermemertrag_electr_heiz_tag_kwh_flag
    type: bool
    restore_value: no 

  - id: waermemertrag_ww_total_kWh_float
    type: float
    restore_value: no
  - id: waermemertrag_ww_total_kWh_flag
    type: bool
    restore_value: no
  - id: waermemertrag_ww_total_mWh
    type: float
    restore_value: no
  - id: waermemertrag_ww_total_mWh_flag
    type: bool
    restore_value: no

  - id: waermemertrag_heiz_total_kWh_float
    type: float
    restore_value: no
  - id: waermemertrag_heiz_total_kWh_flag
    type: bool
    restore_value: no
  - id: waermemertrag_heiz_total_mWh
    type: float
    restore_value: no
  - id: waermemertrag_heiz_total_mWh_flag
    type: bool
    restore_value: no

  - id: waermemertrag_electr_heiz_total_kWh_float
    type: float
    restore_value: no
  - id: waermemertrag_electr_heiz_total_kWh_flag
    type: bool
    restore_value: no
  - id: waermemertrag_electr_heiz_total_mWh
    type: float
    restore_value: no
  - id: waermemertrag_electr_heiz_total_mWh_flag
    type: bool
    restore_value: no

  - id: waermemertrag_electr_ww_total_kWh_float
    type: float
    restore_value: no
  - id: waermemertrag_electr_ww_total_kWh_flag
    type: bool
    restore_value: no
  - id: waermemertrag_electr_ww_total_mWh
    type: float
    restore_value: no
  - id: waermemertrag_electr_ww_total_mWh_flag
    type: bool
    restore_value: no



#request of sensor state by executing lambda-commands. send_state is set to the request-packet. After that update_sensor is activated which sends the command via CAN and gets inactive again
#Abfrage des Sensorstatus durch Ausführen des Lambda-Befehls. send_state wird auf das request-Paket gesetzt. Anschließend wird Update_sensor aktiviert, der den Befehl via CAN absetzt und wieder deaktiviert

#Outside temperature
sensor:
  - platform: template
    name: "Außentemperatur"
    id: temperature_outside
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-lines"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x00;id(send_state)[4]=0x0c;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 10min

#Source temperature
  - platform: template
    name: "Quellentemperatur"
    id: temperature_source
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-lines"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x01;id(send_state)[4]=0xd4;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 10min
    


  - platform: template
    name: "Warmwassertemperatur"
    id: temperature_water
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-lines"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0x0e;id(send_state)[3]=0x01;id(send_state)[4]=0x00;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 10min


  - platform: template
    name: "Verdichterstarts"
    id: VD_starts
    unit_of_measurement: "a.u."
    icon: "mdi:chart-bell-curve-cumulative"
    device_class: "power_factor"
    state_class: "measurement"
    lambda: |-

      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x07;id(send_state)[4]=0x1c;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
    
    
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x07;id(send_state)[4]=0x1d;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);

      if (id(VD_starts_t>0) and id(VD_starts_h>0)){
        float VD_starts_float = id(VD_starts_h)+id(VD_starts_t);
        return VD_starts_float;
      }
      else {return {};}
    update_interval: 5h
    accuracy_decimals: 0



#Calculate coefficient of performance from power sensors for heater, warm-water and used power
#Berechnung COP erfolgt über Sensoren für Stromverbrauch und Wärmeproduktion - kein aktiver Update-Befehl

  - platform: template
    name: "COP-Wert Heizung"
    id: cop_heater
    unit_of_measurement: "a.u."
    icon: "mdi:chart-bell-curve-cumulative"
    device_class: "power_factor"
    state_class: "measurement"
    accuracy_decimals: 2
    lambda: |-
      id(total_electric_energy_heating).update();
      id(total_heating_energy).update();
      id(total_electric_heating_energy).update();
      float heat_cop_float = (id(waermemertrag_heiz_total_mWh)+id(waermemertrag_electr_heiz_total_mWh))/id(el_aufnahmeleistung_heiz_total_mWh);
      return heat_cop_float;
    force_update: true


  - platform: template
    name: "COP-Wert Warmwasser"
    id: cop_water
    unit_of_measurement: "a.u."
    icon: "mdi:chart-bell-curve-cumulative"
    device_class: "power_factor"
    state_class: "measurement"
    accuracy_decimals: 2
    lambda: |-
      id(total_heating_energy_water).update();
      id(total_electric_energy_water).update();
      id(total_heating_energy_water).update();
      float ww_cop_float = (id(waermemertrag_ww_total_mWh)+id(waermemertrag_electr_ww_total_mWh))/id(el_aufnahmeleistung_ww_total_mWh);
      return ww_cop_float;
    force_update: true
    
  - platform: template
    name: "COP-Wert Gesamt"
    id: cop_total
    unit_of_measurement: "a.u."
    icon: "mdi:chart-bell-curve-cumulative"
    device_class: "power_factor"
    state_class: "measurement"
    accuracy_decimals: 2
    lambda: |-
      id(cop_water).update();
      id(cop_heater).update();
      float total_cop_float = ((id(waermemertrag_heiz_total_mWh)+id(waermemertrag_electr_heiz_total_mWh))+(id(waermemertrag_ww_total_mWh)+id(waermemertrag_electr_ww_total_mWh)))/(id(el_aufnahmeleistung_heiz_total_mWh)+id(el_aufnahmeleistung_ww_total_mWh));
      return total_cop_float;
    force_update: true
    update_interval: 6h




  - platform: template
    name: "Rücklauftemperatur Heizung"
    id: temperature_return
    unit_of_measurement: "°C"
    icon: "mdi:waves-arrow-left"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x00;id(send_state)[4]=0x16;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 5min



  - platform: template
    name: "T Heizkreis IST"
    id: t_heizkreis_ist
    unit_of_measurement: "°C"
    icon: "mdi:waves-arrow-right"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x02;id(send_state)[4]=0xca;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 5min


    
  - platform: template
    name: "T Heizkreis Soll"
    id: t_heizkreis_soll
    unit_of_measurement: "°C"
    icon: "mdi:waves-arrow-left"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x01;id(send_state)[4]=0xd7;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 15min


  - platform: template
    name: "Speicher Soll Temperatur"
    id: t_ww_soll
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-water"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0x03;id(send_state)[3]=0x00;id(send_state)[4]=0x00;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 15min


  - platform: template
    name: "Speicher IST Temperatur"
    id: t_ww_ist
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-lines"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1




  - platform: template
    name: "Eco Speicher Soll Temperatur"
    id: ww_temp_eco_log
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-low"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x0a;id(send_state)[4]=0x06;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 10min




  - platform: template
    name: "Komfort Speicher Soll Temperatur"
    id: ww_temp_komfort_log
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-high"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0x13;id(send_state)[3]=0x00;id(send_state)[4]=0x00;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 10min




  - platform: template
    name: "Volumenstrom"
    id: volumenstrom_log
    unit_of_measurement: "l/min"
    icon: "mdi:waves-arrow-right"
    state_class: "measurement"
    accuracy_decimals: 2
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x06;id(send_state)[4]=0x73;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 1min



  - platform: template
    name: "Heizungsdruck"
    id: heizungsdruck_log
    unit_of_measurement: "bar"
    icon: "mdi:gauge"
    device_class: "pressure"
    state_class: "measurement"
    accuracy_decimals: 2
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x06;id(send_state)[4]=0x74;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      return {};
    update_interval: 7min



  - platform: template
    name: "Puffertemperatur"
    id: puffertemperatur_log
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-high"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1


#Automatische Updates durch FEK
  - platform: template
    name: "Luftfeuchtigkeit Wohnraum"
    id: humidity_inside
    unit_of_measurement: "%rH"
    icon: "mdi:water-percent"
    device_class: "humidity"
    state_class: "measurement"
    accuracy_decimals: 1


  - platform: template
    name: "Temperatur Wohnraum"
    id: temperature_inside
    unit_of_measurement: "°C"
    icon: "mdi:thermometer-lines"
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 1
    



  - platform: template
    name: "Stromverbrauch Warmwasser heute"
    id: daily_electric_energy_water
    unit_of_measurement: "kWh"
    device_class: "energy"
    state_class: "measurement"
    accuracy_decimals: 3
    icon: "mdi:transmission-tower"
    lambda: |-
      //el. Leistungsaufnahme WW Tag Wh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x1a;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //el. Leistungsaufnahme WW Tag kWh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x1b;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      if (id(el_aufnahmeleistung_ww_tag_kwh_flag) and id(el_aufnahmeleistung_ww_tag_wh_flag)){
        id(el_aufnahmeleistung_ww_tag_kwh) += id(el_aufnahmeleistung_ww_tag_wh_float);
        float daily_electric_energy_water=id(el_aufnahmeleistung_ww_tag_kwh);
        id(el_aufnahmeleistung_ww_tag_kwh_flag)=false;
        id(el_aufnahmeleistung_ww_tag_wh_flag)=false;
        return daily_electric_energy_water;
        }
      else{return {};}
    update_interval: 15min

  - platform: template
    name: "WM Heizung heute"
    id: daily_heating_energy
    unit_of_measurement: "kWh"
    device_class: "energy"
    icon: "mdi:water-boiler"
    state_class: "measurement"
    accuracy_decimals: 3
    lambda: |-
      //WM Heizen Tag wh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x2e;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //WM Heizen Tag kwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x2f;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      if (id(waermemertrag_heiz_tag_kwh_flag) and id(waermemertrag_heiz_tag_wh_flag)){
        id(waermemertrag_heiz_tag_kwh) += id(waermemertrag_heiz_tag_wh_float);
        float daily_heating_energy=id(waermemertrag_heiz_tag_kwh);
        id(waermemertrag_heiz_tag_kwh_flag)=false;
        id(waermemertrag_heiz_tag_wh_flag)=false;
        return daily_heating_energy;
        }
        else {
          return {};
        }
    update_interval: 15min


  - platform: template
    name: "Stromverbrauch Heizung heute"
    id: daily_electric_energy_heating
    unit_of_measurement: "kWh"
    device_class: "energy"
    state_class: "measurement"
    icon: "mdi:transmission-tower"
    accuracy_decimals: 3   
    lambda: |-
      //el. Leistungsaufnahme Heizen Tag Wh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x1e;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //el. Leistungsaufnahme Heizen Tag kWh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x1f;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      if (id(el_aufnahmeleistung_heiz_tag_kwh_flag) and id(el_aufnahmeleistung_heiz_tag_wh_flag)){
        id(el_aufnahmeleistung_heiz_tag_kwh) += id(el_aufnahmeleistung_heiz_tag_wh_float);
        float daily_electric_energy_heating=id(el_aufnahmeleistung_heiz_tag_kwh);
        id(el_aufnahmeleistung_heiz_tag_kwh_flag)=false;
        id(el_aufnahmeleistung_heiz_tag_wh_flag)=false;
        return daily_electric_energy_heating;
      }
      else{return {};}
    update_interval: 6h


  - platform: template
    name: "WM Warmwasser heute"
    id: daily_heating_energy_water
    unit_of_measurement: "kWh"
    device_class: "energy"
    icon: "mdi:water-boiler"
    state_class: "measurement"
    accuracy_decimals: 3
    lambda: |-
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x2a;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x2b;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      if (id(waermemertrag_ww_tag_kwh_flag) and id(waermemertrag_ww_tag_wh_flag)){
      id(waermemertrag_ww_tag_kwh) += id(waermemertrag_ww_tag_wh_float);
      float daily_heating_energy_water=id(waermemertrag_ww_tag_kwh);
      id(waermemertrag_ww_tag_kwh_flag)=false;
      id(waermemertrag_ww_tag_wh_flag)=false;
      return daily_heating_energy_water;
      }
      else{ return {};
      }
    update_interval: 15min



  - platform: template
    name: "Stromverbrauch Warmwasser total"
    id: total_electric_energy_water
    unit_of_measurement: "MWh"
    device_class: "energy"
    state_class: "total_increasing"
    icon: "mdi:transmission-tower"
    accuracy_decimals: 3
    lambda: |-
      //el. Leistungsaufnahme WW Summe kwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x1c;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //el. Leistungsaufnahme WW Summe Mwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x1d;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      if (id(el_aufnahmeleistung_ww_total_mWh_flag) and id(el_aufnahmeleistung_ww_total_kWh_flag)){
        id(el_aufnahmeleistung_ww_total_mWh) += id(el_aufnahmeleistung_ww_total_kWh_float);
        float total_electric_energy_water=id(el_aufnahmeleistung_ww_total_mWh);
        id(el_aufnahmeleistung_ww_total_mWh_flag)=false;
        id(el_aufnahmeleistung_ww_total_kWh_flag)=false;
        return total_electric_energy_water;
        }
      else {  return {};}



  - platform: template
    name: "Stromverbrauch Heizung total"
    id: total_electric_energy_heating
    unit_of_measurement: "MWh"
    device_class: "energy"
    icon: "mdi:transmission-tower"
    state_class: "total_increasing"
    accuracy_decimals: 3
    lambda: |-
      //el. Leistungsaufnahme Heizen Summe kwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x20;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //el. Leistungsaufnahme Heizen Summe Mwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x21;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      if (id(el_aufnahmeleistung_heiz_total_mWh_flag) and id(el_aufnahmeleistung_heiz_total_kWh_flag)){
        id(el_aufnahmeleistung_heiz_total_mWh) += id(el_aufnahmeleistung_heiz_total_kWh_float);
        float total_electric_energy_heating=id(el_aufnahmeleistung_heiz_total_mWh);
        id(el_aufnahmeleistung_heiz_total_mWh_flag)=false;
        id(el_aufnahmeleistung_heiz_total_mWh_flag)=false;
        return total_electric_energy_heating;
        }
      else {return {};
        
      }



  - platform: template
    name: "WM Heizen total"
    id: total_heating_energy
    unit_of_measurement: "MWh"
    device_class: "energy"
    icon: "mdi:water-boiler"
    state_class: "total_increasing"
    accuracy_decimals: 3
    lambda: |-
      //WM Heizen Summe kwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x30;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //WM Heizen Summe Mwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x31;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);

      //Überprüfung ob beide Leistungswerte empfangen wurden
      if (id(waermemertrag_heiz_total_kWh_flag) and id(waermemertrag_heiz_total_mWh_flag)){
        id(waermemertrag_heiz_total_mWh) += id(waermemertrag_heiz_total_kWh_float);
        float total_heating_energy=id(waermemertrag_heiz_total_mWh);
        id(waermemertrag_heiz_total_kWh_flag)=false;
        id(waermemertrag_heiz_total_mWh_flag)=false;
        return total_heating_energy;
        }
      else {
        return {};
      }



  - platform: template
    name: "WM Warmwasser total"
    id: total_heating_energy_water
    unit_of_measurement: "MWh"
    device_class: "energy"
    icon: "mdi:water-boiler"
    state_class: "total_increasing"
    accuracy_decimals: 3
    lambda: |-
      //WM WW Summe kwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x2c;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //WM WW Summe Mwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x2d;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      //Überprüfung ob beide Leistungswerte empfangen wurden
      if (id(waermemertrag_ww_total_mWh_flag) and id(waermemertrag_ww_total_kWh_flag)){
        id(waermemertrag_ww_total_mWh) += id(waermemertrag_ww_total_kWh_float);
        float total_heating_energy_water=id(waermemertrag_ww_total_mWh);
        id(waermemertrag_ww_total_mWh_flag)=false;
        id(waermemertrag_ww_total_kWh_flag)=false;
        return total_heating_energy_water;
        }
      else {
        return {};
        
      }


  - platform: template
    name: "WM elektr. Warmwasser total"
    id: total_electric_heating_energy_water
    unit_of_measurement: "kWh"
    device_class: "energy"
    icon: "mdi:water-boiler"
    state_class: "total_increasing"
    accuracy_decimals: 3
    lambda: |-
      //WM NE WW Summe kwh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x24;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //WM NE WW Summe MWh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x25;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //Überprüfung ob beide Leistungswerte empfangen wurden
      if (id(waermemertrag_electr_ww_total_mWh_flag) and id(waermemertrag_electr_ww_total_kWh_flag)){
        id(waermemertrag_electr_ww_total_mWh) += id(waermemertrag_electr_ww_total_kWh_float);
        float total_electric_heating_energy_water=id(waermemertrag_electr_ww_total_mWh);
        id(waermemertrag_electr_ww_total_mWh_flag)=false;
        id(waermemertrag_electr_ww_total_kWh_flag)=false;
        return total_electric_heating_energy_water;
      }
      else {
        return {};
        
      }



  - platform: template
    name: "WM elektr. heizen total"
    id: total_electric_heating_energy
    unit_of_measurement: "kWh"
    device_class: "energy"
    icon: "mdi:water-boiler"
    state_class: "total_increasing"
    accuracy_decimals: 3
    lambda: |-
      //WM NE Heizen Summe kWh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x28;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //WM NE Heizen Summe MWh
      id(send_state)[0]=id(PumpCANread_id)[0];id(send_state)[1]=id(PumpCANread_id)[1];id(send_state)[2]=0xfa;id(send_state)[3]=0x09;id(send_state)[4]=0x29;id(send_state)[5]=0x00;id(send_state)[6]=0x00;
      id(update_sensor).publish_state(true);
      id(update_sensor).publish_state(false);
      
      //Überprüfung ob beide Leistungswerte empfangen wurden
      if (id(waermemertrag_electr_heiz_total_kWh_flag) and id(waermemertrag_electr_heiz_total_mWh_flag)){
        id(waermemertrag_electr_heiz_total_mWh) += id(waermemertrag_electr_heiz_total_kWh_float);
        float total_electric_heating_energy=id(waermemertrag_electr_heiz_total_mWh);
        id(waermemertrag_electr_heiz_total_kWh_flag)=false;
        id(waermemertrag_electr_heiz_total_mWh_flag)=false;
        return total_electric_heating_energy;
      }

      else {
        return {};
        
      }


text_sensor:
#Text sensor to change comfort temperature of warm water for automation in homeassistant with photovoltaic
#Sensor zum verändern der Warmwasser-Komfort-Temperatur für Automatisierung mit PV-Anlage
  - platform: homeassistant
    name: "ww_komfort_temp"
    entity_id: input_text.ww_komfort_temp
    id: HASSeingabe_wwkomforttemp
    filters:
     - lambda: |-
          int eingabe=atoi(x.c_str());
          if (eingabe < 250 or eingabe > 600) {
            //Einfache Abfrage der Temperatur durchführen, falls keine gültige Eingabe erfolgt ist   
            id(send_state)[0]=id(PumpCANread_id)[0];
            id(send_state)[1]=id(PumpCANread_id)[1];
            id(send_state)[2]=0x13;
            id(send_state)[3]=0x00;
            id(send_state)[4]=0x00;
            id(send_state)[5]=0x00;
            id(send_state)[6]=0x00;
            return x;


          } else {
            //Wenn User-Eingabe gültig war, Daten für Übertragung an Heizung bereit machen
            id(send_state)[0]=id(PumpCANwrite_id)[0];
            id(send_state)[1]=id(PumpCANwrite_id)[1];
            id(send_state)[2]=0x13;
            id(send_state)[3]=eingabe>>8;
            id(send_state)[4]=eingabe-((eingabe>>8)<<8);
            id(send_state)[5]=0x00;
            id(send_state)[6]=0x00;
            return x;
            
          }
    on_value:
          then:
            - lambda: |-
                //Daten senden
                id(update_sensor).publish_state(true);
                id(update_sensor).publish_state(false);


#Text sensor to change eco temperature of warm water for automation in homeassistant with photovoltaic
#Sensor zum verändern der Warmwasser-Eco-Temperatur für Automatisierung mit PV-Anlage
  - platform: homeassistant
    name: "ww_eco_temp"
    entity_id: input_text.ww_eco_temp
    id: HASSeingabe_wwecotemp
    filters:
     - lambda: |-
          int eingabe=atoi(x.c_str());
          if (eingabe < 250 or eingabe > 600) {

            id(send_state)[0]=id(PumpCANread_id)[0];
            id(send_state)[1]=id(PumpCANread_id)[0];
            id(send_state)[2]=0xfa;
            id(send_state)[3]=0x0a;
            id(send_state)[4]=0x06;
            id(send_state)[5]=0x00;
            id(send_state)[6]=0x00;
            return x;


          } else {

            id(send_state)[0]=id(PumpCANwrite_id)[0];
            id(send_state)[1]=id(PumpCANwrite_id)[1];
            id(send_state)[2]=0xfa;
            id(send_state)[3]=0x0a;
            id(send_state)[4]=0x06;
            id(send_state)[5]=eingabe>>8;
            id(send_state)[6]=eingabe-((eingabe>>8)<<8);
            return x;
            
          }
    on_value:
          then:
            - lambda: |-
                id(update_sensor).publish_state(true);
                id(update_sensor).publish_state(false);



#sensor to send can commands from lambda routines at certain intervals
#Sensor zum Senden von CAN-Befehlen aus Lambda-Routinen
  - platform: template
    id: update_sensor
    on_press:
      then:
        - canbus.send: 
            data: !lambda
              return {(uint8_t) id(send_state)[0],(uint8_t) id(send_state)[1],(uint8_t) id(send_state)[2],(uint8_t) id(send_state)[3], (uint8_t) id(send_state)[4],(uint8_t) id(send_state)[5],(uint8_t) id(send_state)[6]};
            can_id: 0x680
            



button:

#Button to show can command in log - no signal over can bus, but refresh of some sensors is initiated
#Button für CAN-Befehl im Log anzeigen - CAN-Befehl aus Home-Assistant-Dienst - es wird dabei kein Signal an den CAN-Bus gesendet; Führt auch erstmaliges refresh einzelner Sensoren aus
  - platform: template
    name: "Befehl anzeigen / Sensorupdate"
    id: can_befehl_anzeigen
    on_press:
      then:      
        lambda: |-
         id(VD_starts).update();
         id(cop_total).update();
         ESP_LOGI("main", "Value of my hex_sensor: %x, %x, %x, %x, %x, %x, %x", id(sh_state)[0],id(sh_state)[1],id(sh_state)[2],id(sh_state)[3],id(sh_state)[4],id(sh_state)[5],id(sh_state)[6]);


#Button to push can command from home-assistant service
#Button für CAN-Befehl absetzen - CAN-Befehl aus Home-Assistant-Dienst wird an CAN-Bus übermittelt
  - platform: template
    name: CAN-Befehl absetzen
    id: can_send
    # Optional variables:
    icon: "mdi:emoticon-outline"
    on_press:
      then:
        - canbus.send: 
            data: !lambda
              return {(uint8_t) id(sh_state)[0],(uint8_t) id(sh_state)[1],(uint8_t) id(sh_state)[2],(uint8_t) id(sh_state)[3], (uint8_t) id(sh_state)[4],(uint8_t) id(sh_state)[5],(uint8_t) id(sh_state)[6]};
            can_id: 0x680


#Button to reset ESP-device
#Button zum automatisierten reset des ESP-Device
  - platform: restart
    name: "Heizraum ESP restart"
    id: esp_heizraum_restart_bt
    on_press:
      - logger.log: "Button pressed"




spi:
  id: McpSpi
  clk_pin: <clk_pin>
  mosi_pin: <mosi_pin>
  miso_pin: <miso_pin>



##################################################################################################################
#Eventually the CAN_ID of the esp-device must be changed
#additionally don't forget to change the value of the variable internalResponse_id in the global variable section
##################################################################################################################

canbus:
  - platform: mcp2515
    id: my_mcp2515
    spi_id: McpSpi
    cs_pin: GPIO17
    can_id: 680
    use_extended_id: false
    bit_rate: 20kbps
    on_frame:
##################################################################################################################

#compressor Starts thousands
    - can_id: 0x180
      then:
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0xfa and x[3]==0x07 and x[4]==0x1c) {
                float VD_x =float((int16_t((x[6])+( (x[5])<<8))))*1000;
                id(VD_starts_t)=VD_x;
                ESP_LOGD("main", "Verdichter Starts 1000 empfangen over can is %f", VD_x);
              }

#compressor Starts hundreds
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0xfa and x[3]==0x07 and x[4]==0x1d) {
                int VD_x =((int16_t((x[6])+( (x[5])<<8))));
                id(VD_starts_h)=VD_x;
                ESP_LOGD("main", "Verdichter Starts 100 empfangen over can is %i", VD_x);
              }

#T Heizkreis WW Komfort Soll Wert
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0x13) {
              float temperature =(float((int16_t((x[4])+( (x[3])<<8))))/10);
              id(ww_temp_komfort_log).publish_state(temperature);
              ESP_LOGD("main", "T Komfort Soll empfangen over can is %f", temperature);
            }
#Volumenstrom (l/min)
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0xfa and x[3]==0x06 and x[4]==0x73) {
              float current =(float((int16_t((x[6])+( (x[5])<<8))))/100);
              id(volumenstrom_log).publish_state(current);
              id(volumenstrom_float)=current;
              ESP_LOGD("main", "l/min Volumenstrom empfangen over can is %f", current);
            }

#Heizungsdruck (bar)
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0xfa and x[3]==0x06 and x[4]==0x74) {
              float pressure =(float((int16_t((x[6])+( (x[5])<<8))))/100);
              id(heizungsdruck_log).publish_state(pressure);
              ESP_LOGD("main", "bar Heizungsdruck empfangen over can is %f", pressure);
            }


#T Heizkreis WW Eco Soll Abfrage
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[3]==0x0a and x[4]==0x06) {
              float temperature =(float((int16_t((x[6])+( (x[5])<<8))))/10);
              id(ww_temp_eco_log).publish_state(temperature);
              ESP_LOGD("main", "T Eco Soll empfangen over can is %f", temperature);
            }


#T Heizkreis IST Abfrage
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[3]==0x02 and x[4] == 0xca) {
              float temperature =(float((int16_t((x[6])+( (x[5])<<8))))/10);
              id(t_heizkreis_ist).publish_state(temperature);
              ESP_LOGD("main", "T Heizkreis IST empfangen over can is %f", temperature);
            }
#T Heizkreis Soll Abfrage
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[3]==0x01 and x[4] == 0xd7) {
              float temperature =(float((int16_t((x[6])+( (x[5])<<8))))/10);
              id(t_heizkreis_soll).publish_state(temperature);
              ESP_LOGD("main", "T Heizkreis Soll empfangen over can is %f", temperature);
            }
#T WW Soll Abfrage
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0x03) {
              float temperature =(float((int16_t((x[4])+( (x[3])<<8))))/10);
              id(t_ww_soll).publish_state(temperature);
              ESP_LOGD("main", "T Warmwasser Soll empfangen over can is %f", temperature);
            }


#Warmwasser-Temperaturabfrage + Gerätespezifischer Offset 3.9 °C
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[3]==0x00 and x[4] == 0x0e) {
              float temperature =(float((int16_t((x[6])+( (x[5])<<8))))/10)+3.9;
              id(temperature_water).publish_state(temperature);
              ESP_LOGD("main", "Warmwasser-Temperature empfangen over can is %f", temperature);
            }
            
#Quellen-Temperatur
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[3]==0x01 and x[4] == 0xd4) {
              float temperature =float((int16_t((x[6])+( (x[5])<<8))))/10;
              id(temperature_source).publish_state(temperature);
              ESP_LOGD("main", "Quellen-Temperature received over can is %f", temperature);
            }

#Speicher IST-temperatur
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0x0e) {
              float temperature =float((int16_t((x[4])+( (x[3])<<8))))/10;
              id(t_ww_ist).publish_state(temperature);
              ESP_LOGD("main", "Speicher-Temperature received over can is %f", temperature);
            }

#Rücklauftemperatur
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[3]==0x00 and x[4] == 0x16) {
              float temperature =float((int16_t((x[6])+( (x[5])<<8))))/10;
              id(temperature_return).publish_state(temperature);
              ESP_LOGD("main", "Rücklauf-Temperature received over can is %f", temperature);
            }


#Außentemperatur
#float temperature =float(float((int((x[6])+( (x[5])<<8))))/10);
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[3]==0x00 and x[4] == 0x0c) {
              float temperature =float((int16_t((x[6])+( (x[5])<<8))));
              if (temperature > 65000){
                temperature=(temperature-65536);
              }
              temperature=temperature/10;
              id(temperature_outside).publish_state(temperature);
              ESP_LOGD("main", "Aussen-Temperature received over can is %f", temperature);
            }
#Elektrische Leistungsaufnahme Wh /kWh
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0xfa and x[3]==0x09) {
              if (x[4]==0x1a){
                id(el_aufnahmeleistung_ww_tag_wh_float) = (float((int((x[6])+( (x[5])<<8))))/1000);
                id(el_aufnahmeleistung_ww_tag_wh_flag)=true;
                ESP_LOGD("main", "el_aufnahmeleistung_ww_tag_kwh received over can is %f", id(el_aufnahmeleistung_ww_tag_wh_float));}
              else if (x[4]==0x1e){
                id(el_aufnahmeleistung_heiz_tag_wh_float) = (float((int((x[6])+( (x[5])<<8))))/1000);
                id(el_aufnahmeleistung_heiz_tag_wh_flag) = true;
                ESP_LOGD("main", "el_aufnahmeleistung_heiz_tag_wh received over can is %f", id(el_aufnahmeleistung_heiz_tag_wh_float));}
              else if (x[4]==0x1c){
                id(el_aufnahmeleistung_ww_total_kWh_float) = (float((int((x[6])+( (x[5])<<8))))/1000);
                id(el_aufnahmeleistung_ww_total_kWh_flag)=true;
                ESP_LOGD("main", "el_aufnahmeleistung_ww_total_kWh received over can is %f", id(el_aufnahmeleistung_ww_total_kWh_float));}
              else if (x[4]==0x20){
                id(el_aufnahmeleistung_heiz_total_kWh_float) = (float((int((x[6])+( (x[5])<<8))))/1000);
                id(el_aufnahmeleistung_heiz_total_kWh_flag) = true;
                ESP_LOGD("main", "el_aufnahmeleistung_heiz_total_kWh received over can is %f", id(el_aufnahmeleistung_heiz_total_kWh_float));}
              }

#Elektrische Leistungsaufnahme kWh / MWH
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0xfa and x[3]==0x09) {
              if(x[4]==0x1b){
              id(el_aufnahmeleistung_ww_tag_kwh) =float(int((x[6])+( (x[5])<<8)));
              id(el_aufnahmeleistung_ww_tag_kwh_flag)=true;
              ESP_LOGD("main", "el_aufnahmeleistung_ww_tag_kwh received over can is %f", id(el_aufnahmeleistung_ww_tag_kwh));}
              else if(x[4]==0x1f){
                id(el_aufnahmeleistung_heiz_tag_kwh) =float(int((x[6])+( (x[5])<<8)));
                id(el_aufnahmeleistung_heiz_tag_kwh_flag)=true;
                ESP_LOGD("main", "el_aufnahmeleistung_heiz_tag_kwh received over can is %f", id(el_aufnahmeleistung_heiz_tag_kwh));}
              else if(x[4]==0x1d){
              id(el_aufnahmeleistung_ww_total_mWh) =float(int((x[6])+( (x[5])<<8)));
              id(el_aufnahmeleistung_ww_total_mWh_flag)=true;
              ESP_LOGD("main", "el_aufnahmeleistung_ww_total_mWh received over can is %f", id(el_aufnahmeleistung_ww_total_mWh));}
              else if(x[4]==0x21){
                id(el_aufnahmeleistung_heiz_total_mWh) =float(int((x[6])+( (x[5])<<8)));
                id(el_aufnahmeleistung_heiz_total_mWh_flag)=true;
                ESP_LOGD("main", "el_aufnahmeleistung_heiz_total_mWh received over can is %f", id(el_aufnahmeleistung_heiz_total_mWh));}
            }


#Wärmeertrag WW/Heizung MWh / kWH
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0xfa and x[3]==0x09) {
              if(x[4]==0x25){
                id(waermemertrag_electr_ww_total_mWh) =float(int((x[6])+( (x[5])<<8)));
                id(waermemertrag_electr_ww_total_mWh_flag)=true;
                ESP_LOGD("main", "waermemertrag_electr_ww_tag_kwh received over can is %f", id(waermemertrag_electr_ww_total_mWh));}
              else if(x[4]==0x29){
                id(waermemertrag_electr_heiz_total_mWh) =float(int((x[6])+( (x[5])<<8)));
                id(waermemertrag_electr_heiz_total_mWh_flag)=true;
                ESP_LOGD("main", "waermemertrag_electr_heiz_tag_kwh received over can is %f", id(waermemertrag_electr_heiz_total_mWh));}
              else if(x[4]==0x2b){
              id(waermemertrag_ww_tag_kwh) =float(int((x[6])+( (x[5])<<8)));
              id(waermemertrag_ww_tag_kwh_flag)=true;
              ESP_LOGD("main", "waermemertrag_ww_tag_kwh received over can is %f", id(waermemertrag_ww_tag_kwh));}
              else if(x[4]==0x2d){
              id(waermemertrag_ww_total_mWh) =float(int((x[6])+( (x[5])<<8)));
              id(waermemertrag_ww_total_mWh_flag)=true;
              ESP_LOGD("main", "waermemertrag_ww_total_mWh received over can is %f", id(waermemertrag_ww_total_mWh));}
              else if(x[4]==0x2f){
                id(waermemertrag_heiz_tag_kwh) =float(int((x[6])+( (x[5])<<8)));
                id(waermemertrag_heiz_tag_kwh_flag)=true;
                ESP_LOGD("main", "waermemertrag_heiz_tag_kwh received over can is %f", id(waermemertrag_heiz_tag_kwh));}
              else if(x[4]==0x31){
                id(waermemertrag_heiz_total_mWh) =float(int((x[6])+( (x[5])<<8)));
                id(waermemertrag_heiz_total_mWh_flag)=true;
                ESP_LOGD("main", "waermemertrag_heiz_total_kWh_float received over can is %f", id(waermemertrag_heiz_total_mWh));}
            }


#Wärmeertrag WW/Heizung Wh / kWH
        - lambda: |-
            if(x[0]==id(internalResponse_id)[0] and x[1]==id(internalResponse_id)[1] and x[2]==0xfa and x[3]==0x09) {
              if(x[4]==0x24){
                id(waermemertrag_electr_ww_total_kWh_float) =float(int((x[6])+( (x[5])<<8)))/1000;
                id(waermemertrag_electr_ww_total_kWh_flag)=true;
                ESP_LOGD("main", "waermemertrag_electr_ww_tag_wh_float received over can is %f", id(waermemertrag_electr_ww_total_kWh_float));}
              else if(x[4]==0x28){
                id(waermemertrag_electr_heiz_total_kWh_float) =float(int((x[6])+( (x[5])<<8)))/1000;
                id(waermemertrag_electr_heiz_total_kWh_flag)=true;
                ESP_LOGD("main", "waermemertrag_electr_heiz_tag_wh_float received over can is %f", id(waermemertrag_electr_heiz_total_kWh_float));}
              else if(x[4]==0x2a){
              id(waermemertrag_ww_tag_wh_float) =float(int((x[6])+( (x[5])<<8)))/1000;
              id(waermemertrag_ww_tag_wh_flag)=true;
              ESP_LOGD("main", "waermemertrag_ww_tag_wh_float received over can is %f", id(waermemertrag_ww_tag_wh_float));}
              else if(x[4]==0x2c){
              id(waermemertrag_ww_total_kWh_float) =float(int((x[6])+( (x[5])<<8)))/1000;
              id(waermemertrag_ww_total_kWh_flag)=true;
              ESP_LOGD("main", "waermemertrag_ww_total_kWh_float received over can is %f", id(waermemertrag_ww_total_kWh_float));}
              else if(x[4]==0x2e){
                id(waermemertrag_heiz_tag_wh_float) =float(int((x[6])+( (x[5])<<8)))/1000;
                id(waermemertrag_heiz_tag_wh_flag)=true;
                ESP_LOGD("main", "waermemertrag_heiz_tag_wh_float received over can is %f", id(waermemertrag_heiz_tag_wh_float));}
              else if(x[4]==0x30){
                id(waermemertrag_heiz_total_kWh_float) =float(int((x[6])+( (x[5])<<8)))/1000;
                id(waermemertrag_heiz_total_kWh_flag)=true;
                ESP_LOGD("main", "waermemertrag_heiz_total_kWh_float received over can is %f", id(waermemertrag_heiz_total_kWh_float));}
            }





#Read humidity from FEK transmits
    - can_id: 0x301
      then:
        - lambda: |-
            if(x[0]==id(FekCANread_id)[0] and x[1]==id(FekCANread_id)[1] and x[2]==0x75) {
              float humidity =float(float((int16_t((x[4])+( (x[3])<<8))))/10);
              id(humidity_inside).publish_state(humidity);
              ESP_LOGD("main", "Humidity received over can is %f", humidity);
            }
#read room temperature from FEK transmits
        - lambda: |-
            if(x[0]==id(FekCANread_id)[0] and x[1]==id(FekCANread_id)[1] and x[2]==0x11) {
              float temperature =float((int16_t((x[4])+( (x[3])<<8))))/10;
              id(temperature_inside).publish_state(temperature);
              ESP_LOGD("main", "Raum-Temperature received over can is %f", temperature);
            }



#Read other CAN-messages in the bus, that may be interesting and show them in the logs
    - can_id: 0x180
      then:
        - lambda: |-
              int wert0 = int(x[0]);
              int wert1 =int(x[1]);
              int wert2 =int(x[2]);
              int wert3 =int(x[3]);
              int wert4 =int(x[4]);
              int wert5 =int(x[5]);
              int wert6 =int(x[6]);
              float wert7 = float(int((x[6])+( (x[5])<<8)));
              float wert8 = float(int((x[4])+( (x[3])<<8)));
              ESP_LOGI("main", "Antwort von 180 Hex: %x %x %x %x %x %x %x", wert0, wert1, wert2, wert3, wert4, wert5, wert6);
              ESP_LOGI("main", "Antwort von 180 Float: %f", wert7);
              ESP_LOGI("main", "Antwort von 180 Dez.: %i %i", wert5, wert6);
              ESP_LOGI("main", "Antwort klein von 180 Float: %f", wert8);
              ESP_LOGI("main", "Antwort klein von 180 Dez.: %i %i", wert3, wert4);

              
    - can_id: 0x700
      then:
        - lambda: |-
              int wert0 = int(x[0]);
              int wert1 =int(x[1]);
              int wert2 =int(x[2]);
              int wert3 =int(x[3]);
              int wert4 =int(x[4]);
              int wert5 =int(x[5]);
              int wert6 =int(x[6]);
              float wert7 = float(int((x[6])+( (x[5])<<8)));
              float wert8 = float(int((x[4])+( (x[3])<<8)));
              ESP_LOGI("main", "Antwort von 700 Hex: %x %x %x %x %x %x %x", wert0, wert1, wert2, wert3, wert4, wert5, wert6);
              ESP_LOGI("main", "Antwort von 700 Float: %f", wert7);
              ESP_LOGI("main", "Antwort von 700 Dez.: %i %i", wert5, wert6);
              ESP_LOGI("main", "Antwort klein von 700 Float: %f", wert8);
              ESP_LOGI("main", "Antwort klein von 700 Dez.: %i %i", wert3, wert4);

    - can_id: 0x480
      then:
        - lambda: |-
              int wert0 = int(x[0]);
              int wert1 =int(x[1]);
              int wert2 =int(x[2]);
              int wert3 =int(x[3]);
              int wert4 =int(x[4]);
              int wert5 =int(x[5]);
              int wert6 =int(x[6]);
              float wert7 = float(int((x[6])+( (x[5])<<8)));
              float wert8 = float(int((x[4])+( (x[3])<<8)));
              ESP_LOGI("main", "Antwort von 480 Hex: %x %x %x %x %x %x %x", wert0, wert1, wert2, wert3, wert4, wert5, wert6);
              ESP_LOGI("main", "Antwort von 480 Float: %f", wert7);
              ESP_LOGI("main", "Antwort von 480 Dez.: %i %i", wert5, wert6);
              ESP_LOGI("main", "Antwort klein von 480 Float: %f", wert8);
              ESP_LOGI("main", "Antwort klein von 480 Dez.: %i %i", wert3, wert4);

    - can_id: 0x100
      then:
        - lambda: |-
              int wert0 = int(x[0]);
              int wert1 =int(x[1]);
              int wert2 =int(x[2]);
              int wert3 =int(x[3]);
              int wert4 =int(x[4]);
              int wert5 =int(x[5]);
              int wert6 =int(x[6]);
              float wert7 = float(int((x[6])+( (x[5])<<8)));
              float wert8 = float(int((x[4])+( (x[3])<<8)));
              ESP_LOGI("main", "Antwort von 100 Hex: %x %x %x %x %x %x %x", wert0, wert1, wert2, wert3, wert4, wert5, wert6);
              ESP_LOGI("main", "Antwort von 100 Float: %f", wert7);
              ESP_LOGI("main", "Antwort von 100 Dez.: %i %i", wert5, wert6);
              ESP_LOGI("main", "Antwort klein von 100 Float: %f", wert8);
              ESP_LOGI("main", "Antwort klein von 100 Dez.: %i %i", wert3, wert4);

    - can_id: 0x301
      then:
        - lambda: |-
              int wert0 = int(x[0]);
              int wert1 =int(x[1]);
              int wert2 =int(x[2]);
              int wert3 =int(x[3]);
              int wert4 =int(x[4]);
              int wert5 =int(x[5]);
              int wert6 =int(x[6]);
              float wert7 = float(int((x[6])+( (x[5])<<8)));
              float wert8 = float(int((x[4])+( (x[3])<<8)));
              ESP_LOGI("main", "Antwort von 301 Hex: %x %x %x %x %x %x %x", wert0, wert1, wert2, wert3, wert4, wert5, wert6);
              ESP_LOGI("main", "Antwort von 301 Float: %f", wert7);
              ESP_LOGI("main", "Antwort von 301 Dez.: %i %i", wert5, wert6);
              ESP_LOGI("main", "Antwort klein von 301 Float: %f", wert8);
              ESP_LOGI("main", "Antwort klein von 301 Dez.: %i %i", wert3, wert4);

