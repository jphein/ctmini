# ctmini
Open Source Current Meter Project for Home Assistant (ESPHome)
Open Hardware with schematics and gerber files. 
Up to 38 current sensors per board for full electrical panel monitoring. 

I wanted to measure the current of every branch from my main panel, and display the info in Home Assistant. In North America where I live, it is common to have a 200a split phase circuit coming in from the grid. a 120v line a neutral and an inverted 120v line. You get 240v by using the two hots, and you get 120v by using either one of the hots and the neutral. I looked up commercial whole panel energy monitoring kits, but they are super expensive, and often don't have enough current sensors. For my panel I need around 24 seperate sensors to monitor everything. So, I decided to go the diy route. I wanted a lego-like dev board aproach. 

# Micro Controller
I decided to go with the ESP32 for the microcontroller since it supports up to 8 analog inputs, (only 6 are exposed with the common dev board: NodeMCU-32s). It's cheap and available on Aliexpress or Amazon. It also has to I2C channels that can connect to other ADC devices like the ADC1115. The ADC1115 dev barod support 4 anaolog input pins, and 4 different I2c addresses per channel. So you can connect up to 8 of them per esp32 dev board. which supplies up to 32 analog inputs pins in addition to the 6 available on the dev board. 
![image](https://github.com/user-attachments/assets/dc49ffaf-ac4e-4327-b88e-88e1c3f9b8d1)
https://medium.com/@cedricdiego0/esp32-with-an-external-adc-ads1115-a86d3b51bb8

# Current Transformer Sensor (CT Sensor)
CT sensors seem to be pretty expenseive, but I found this sensor on AliExpress: https://s.click.aliexpress.com/e/_DeVVdyx. For $1.29ea.
1pc AC Current transformer CT for Mini ammeter Current meter 0-100A ratio : 1000:1 , XH2.54
![image](https://github.com/user-attachments/assets/c2468191-3f85-44cd-90d5-fee3b56ec6ce)

I found this schematic in the home assistant forums.
![ct-schematic](https://github.com/user-attachments/assets/a160f52a-f98d-4488-a3d9-f8546ad63bfb) from  https://community.home-assistant.io/t/esphome-ct-clamp-30a-1v/284423 

I found this project: https://github.com/danpeig/ESP32EnergyMonitor, based on this one: https://github.com/openenergymonitor/EmonLib
This is almost what I want, but the prototype is using prototype board. I wanted printed circuit boards.
![image](https://github.com/user-attachments/assets/9f3083d0-d7d1-4819-9b5c-ad5e78676555)

https://github.com/danpeig/ESP32EnergyMonitor/blob/main/images/electronic_parts.png
![electronic_parts](https://github.com/user-attachments/assets/a83f339e-b8e2-4af2-8d2c-815835a759d1)

I started with my own prototype to test the ct sensor, and the esphome's native ctsensor component. I used the first schematic. the one in the forum post, and it seemed to work. 
![20240902_113539](https://github.com/user-attachments/assets/80766e44-1a8f-4789-ad2a-26e74f23254c)
```
#ctmini.yaml
esphome:
  name: current-sensors
  friendly_name: current sensors

esp32:
  board: nodemcu-32s
  framework:
    type: arduino

# Enable logging
logger:

# then remember to add time, you need this for daily consumption sensors
# add the following:
time:
  - platform: sntp
    id: my_time

sensor:
  - platform: adc
    pin: A0
    id: adc_sensor0
    update_interval: 6s
    attenuation: auto

  - platform: ct_clamp
    sensor: adc_sensor0
    name: "Sensor0 Current"
    update_interval: 10s
    filters: 
      - calibrate_linear: 
        - 0.00773 -> 0
        - 0.09481 -> 0.04
        - 1.07216 -> 0.80
    id: sensor0
    internal: True

  - platform: template				# this will be the final current reading visible in the front end		
    name: "Sensor0 Current"            
    id: current0
    lambda: |-					        
      if (id(sensor0).state > 0.01){ 
        return (id(sensor0).state);
      } else {
        return 0.0;
      }
    device_class: current
    update_interval: 5s
    accuracy_decimals: 2			
    unit_of_measurement: A  

  - platform: total_daily_energy #this will calculate the total current used for the day (AmpHour)
    name: "Sensor0 Daily Current"
    power_id: current0
    accuracy_decimals: 2
    unit_of_measurement: Ah

  - platform: template				#this sensor will measure your power0 consumption (Watt)
    name: "Sensor0 Measured Power"            
    id: power0
    lambda: |-
      return id(current0).state * 120;
    accuracy_decimals: 2
    update_interval: 5s
    device_class: power
    unit_of_measurement: W

  - platform: total_daily_energy  		#this will keep track of power0 consumed daily (kwh)      
    name: "Sensor0 Total Daily Power Consumption"
    unit_of_measurement: kWh
    power_id: power0
    accuracy_decimals: 2
    filters:
      - multiply: 0.001
    device_class: energy
```
![image](https://github.com/user-attachments/assets/79098e75-5fd0-4e3d-b6e2-e429612ea5e1)
![20240902_133311](https://github.com/user-attachments/assets/ac521749-3a84-4242-8aa9-efcfaae87dba)
![Screenshot from 2024-09-02 14-46-53](https://github.com/user-attachments/assets/15ddaa2d-8c4b-49c8-9204-a49b5fb8423f)
![Screenshot from 2024-09-02 14-49-52](https://github.com/user-attachments/assets/8b25f68a-5d98-4a54-817c-0537f010dbc2)
![Screenshot from 2024-09-02 14-53-53](https://github.com/user-attachments/assets/09b89304-b2cd-4ca8-83bd-ca8d2e85ca6d)
![image](https://github.com/user-attachments/assets/3b02087b-465a-4353-ad68-c532d93c0285)
![image](https://github.com/user-attachments/assets/d2e487e4-97d4-434b-88ff-f5b23ce2c044)

Then I made some little buscards:
![image](https://github.com/user-attachments/assets/dcfaf90e-e9ba-4d01-bcdc-b907c7e19a70)
![image](https://github.com/user-attachments/assets/79c60ead-468c-40c8-be9e-6d3ca35b6c89)

I decided to use these little screw down terminals for all the connectors. They seem sturdy enough if you use solid copper, or add solder or ferrules to stranded copper. 
![1000000815](https://github.com/user-attachments/assets/8a335231-9ee4-48ba-9f4e-841fa707b112)

Here is my idea
![1000000813](https://github.com/user-attachments/assets/f1fd3168-fe88-4c47-9a29-0c5370b4886c)
