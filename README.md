# CSUS Competitive Robotics Club AUV PCB Descriptions

CSUS Competitive Robotics Club AUV PCB. This only includes the pinnouts and descriptions of the boards. 

Anything written below is not helpful unless you are looking for electrical system specific information. 

Note that the Main board and Secondary board have SHARED grounds, they will be connected using the mounting holes and a spade connector. They communicate with 5 pin JST connectors. Main board operates with a 2S lipo battery, while the secondary thruster PCB operates with a 4S battery. The hydrophone PCB will have the same ground as the 2S PCB, and will be powered with 3.3V. The images may NOT be the same as what is actually in the design files, however the differences will be minor, like silkscreen, pin placements, etc. Any major improvements will have updated pictures. 

MAIN BOARD SCHEMATIC AND BOARD LAYOUT INFORMATION: 

(Ideally open the schematic and board layout PDF while reading the information below)

First I just want to say that the reason there is B_EN (buck converter enable) is that for our RoboSub competition we require a on off killswitch, so the EN pin will feed to the secondary board that will be connected to a P channel mosfet gate driver and the logic needed to be reversed for the buck converter, optionally we dont need to turn off our electronics however we decided to make the killswitch turn everything off, I added three options with 0 ohm resistors so we can adjust this later, we can either use B_EN, or EN, or just 5V so we can decide later too. Notibly, the 5V for the killswitch/water detection is different from the 5V for the other electronics, this is because if you use the same 5V, then it will be in a switching state and will not be stable meaning it will not either be on or off. 

Ill start from the top left of the schematic and work my way to the bottom right.

Water Detection, Killswitch, and 7.4V to 5V LDO:

The top left we have a LDO, that converts 7.4 to 5V for the killswitch and water detection logic. The killswitch is just connected to an external switch. The water detection how it will work is it uses 2.54mm headers that are glued to a sponge. Since this is pretty noisy there are RC filters. When water goes into the sponge, the wires short and will cause the AND gate output (where the EN pin is) to output 5V, which is good for the P channel mosfet driver since Vgs = Vg - Vs and means the mosfet will be off. The logic is inverted to the buck converter since if the EN pin outputs 5V, then we want the buck to be off because this means either water is detected, and or the killswitch is enabled. LEDs are there so we can visually see if they are on or not.
<img width="2005" height="1487" alt="image" src="https://github.com/user-attachments/assets/712bdcb4-b74f-4971-b92c-025a90d681ac" />


5V to 3.3V LDO, and Sensors:

Here we just have a 3 pin header for LED strips which will be used for diagnosing issues, an IMU, and a pressure sensor. The LDO is essentially just used to power the pressure sensor (GY-MS5837-30BA). I also added 100nF decoupling capacitors for transients and zero ohm resistors for adjustability on all the data pins to the teensy.
<img width="2732" height="1552" alt="image" src="https://github.com/user-attachments/assets/429a3998-0f8b-4480-9924-cacabdad60f9" />

7.4V to 5V 20A Buck Converter, Voltage and Current Measurement:

We will likely not be pulling 20A, but we are powering a lot of parts. A Jetson Orin Nano (which to my understanding shouldn't be powered off 5V, minimum 7V, so I might unfortunately need to add an external step up converter, though some people said it worked on 5V so this is TBD), Raspberry pi 5 8gb, and a teensy, some sensors. We also will be powering servos on this rail with an external PWM controller. The output capacitors of the buck converter are really big, I also thought the sizing was incorrect but I rechecked calculations from the datasheet and it seemed right, I also was assuming the worst case, since the size of the caps is pretty large I added an 0603 bleed resistor. We also would like to find out the amount of current everything is pulling so I made a differential amplifier across a shunt resistor to find the current which we can calculate using I = V/R, potentially the resistor values will be adjusted so the op amp wouldnt be a unity gain amp and would be easier to read. The voltage of the battery is simply just with a voltage divider. All of this is using 0 ohm resistors so it can be adjusted later. We also are using XT30 connectors to connect to the Jetson, Pi, and anything else we would want to power, with small 0603 100nF decoupling capacitors for transients.
<img width="2672" height="990" alt="image" src="https://github.com/user-attachments/assets/a9ef61ae-a8f9-4f20-a8bc-1f39b25a447e" />

Teensy 4.1:

Not a whole lot to say, it just connects to 2.54mm pin headers (the indented pins are the ones being used by sensors)
<img width="977" height="1065" alt="image" src="https://github.com/user-attachments/assets/347daa75-7ac8-4f92-86d1-87d8ab7864f8" />

Board Layout is still my area of least expertise and am still trying to learn and improve. So if you see anything that is questionable let me know. I also know that adding traces in the power plane (layer 3) is not ideal, but I think the tradeoff is worth it because of the oz copper I will be trying to use, and routing them on the front or back would make the GND plane to the XT30 connectors basically be "cut" which is bad because of the oz copper. I also added little astricts * so I can identify which pins are connected to something (like sensors), there is a PDF here that will show the pinnout. I also added a lot of test points and labeling. Also the RX and TX next to the teensy are so it can communicate with the pi and jetson orin nano. I separated the ground loops, so the basically the buck converter has one loop back to the battery, and all the rest of the components share the same ground loop, of course while making sure there are no ground islands and that the board follows basic DRC. 

Layers:
Front (red) - Signals and GND Plane

2nd Layer (green) - GND Plane

3rd Layer (orange) - Power Plane

4th Layer (blue) - Signals and GND Plane

Ideally, to cut cost, I am trying to use 1oz for the front and back, and 0.5 oz in the middle layers. 

Board size is 122mm x 71.5mm, radius fillet on the edges are 2mm, and uses M2 screws for mounting holes. 

Minor issues in the layout that you will likely see in the following images that since have been fixed/improved:
*  None Yet Since 1/3/2026

All Planes visible:
<img width="3200" height="1985" alt="image" src="https://github.com/user-attachments/assets/118d0f86-5926-4caf-949a-cf67f70e9a21" />

Front Plane:
<img width="3197" height="1945" alt="image" src="https://github.com/user-attachments/assets/74dcc462-98cd-4b80-b11a-02913cbcea2c" />

2nd Layer:
<img width="3202" height="1925" alt="image" src="https://github.com/user-attachments/assets/433bcdc0-0805-4e0b-85c3-05157cca27e8" />

3rd Layer (again, even though not optimal, a lot of traces are here because if I put them on the top or bottom layer, the ground or power plane will be "cut" and those planes have a higher oz copper meaning they can carry more current):
<img width="3202" height="1915" alt="image" src="https://github.com/user-attachments/assets/e09591eb-0b74-4529-8cf3-f49b98833bfc" />

4th Layer:
<img width="3195" height="1932" alt="image" src="https://github.com/user-attachments/assets/9b44b56c-d637-48b7-9f02-b151c571f2b9" />

3D View (Front):
<img width="3352" height="2120" alt="image" src="https://github.com/user-attachments/assets/aae7dff1-7a4f-4d18-bdbc-8f35af4601b7" />

3D View (Back):
<img width="3350" height="2082" alt="image" src="https://github.com/user-attachments/assets/13c051e9-7a79-46d1-938f-7ca69dc87f80" />
