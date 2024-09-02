# ctmini
Open Source Current Meter Project for Home Assistant (ESPHome)
Open Hardware with schematics and gerber files. 
Up to 38 current sensors per board for full electrical panel monitoring. 

I wanted to measure the current of every branch from my main panel, and display the info in Home Assistant. In North America where I live, it is common to have a 200a split phase circuit coming in from the grid. a 120v line a neutral and an inverted 120v line. You get 240v by using the two hots, and you get 120v by using either one of the hots and the neutral. I looked up commercial whole panel energy monitoring kits, but they are super expensive, and often don't have enough current sensors. For my panel I need around 24 seperate sensors to monitor everything. So, I decided to go the diy route. 

I decided to go with the ESP32 for the microcontroller since it supports up to 8 analog inputs, (only 6 are exposed with the common dev board: NodeMCU-32s). It's cheap and available on Aliexpress or Amazon. It also has to I2C channels that can connect to other ADC devices like the ADC1115. The ADC1115 support 4 anaolog input pins, and 4 different I2c addresses per channel. So you can connect up to 8 of them per esp32 dev board. which supplies up to 32 analog inputs pins in addition to the 6 available on the dev board. 

I found this project https://github.com/openenergymonitor/EmonLibers seem to be pretty expenseive, but I found this sensor on AliExpress: https://s.click.aliexpress.com/e/_DeVVdyx. For $1.29ea.
I found this schematic in the home assistant forums.
![ct-schematic](https://github.com/user-attachments/assets/a160f52a-f98d-4488-a3d9-f8546ad63bfb) from  https://community.home-assistant.io/t/esphome-ct-clamp-30a-1v/284423 

I found this project: https://github.com/danpeig/ESP32EnergyMonitor, based on this one: https://github.com/openenergymonitor/EmonLib
This is almost what I want, but the prototype is using prototype board. I wanted printed circuit boards, and plastic cases.

It all started when 
