# CSUS Competitive Robotics Club AUV PCB Descriptions
CSUS Competitive Robotics Club AUV PCB. This only includes the pinnouts and descriptions of the boards. 



SCHEMATIC AND BOARD LAYOUT INFORMATION: (Ideally open the schematic and board layout PDF while reading the information below, a logo will be added as well but this is purely cosmetic)

SCHEMATIC AND BOARD LAYOUT INFORMATION: 
First I just want to say that the reason there is B_EN (buck converter enable) is that for our RoboSub competition we require a on off killswitch, so the EN pin will feed to the secondary board that will be connected to a P channel mosfet gate driver and the logic needed to be reversed for the buck converter, optionally we dont need to turn off our electronics however we decided to make the killswitch turn everything off, I added three options with 0 ohm resistors so we can adjust this later, we can either use B_EN, or EN, or just 5V so we can decide later too. Notibly, the 5V for the killswitch/water detection is different from the 5V for the other electronics, this is because if you use the same 5V, then it will be in a switching state and will not be stable meaning it will not either be on or off. 

Ill start from the top left of the schematic and work my way to the bottom right.

Water Detection, Killswitch, and 7.4V to 5V LDO:

The top left we have a LDO, that converts 7.4 to 5V for the killswitch and water detection logic. The killswitch is just connected to an external switch. The water detection how it will work is it uses 2.54mm headers that are glued to a sponge. Since this is pretty noisy there are RC filters. When water goes into the sponge, the wires short and will cause the AND gate output (where the EN pin is) to output 5V, which is good for the P channel mosfet driver since Vgs = Vg - Vs and means the mosfet will be off. The logic is inverted to the buck converter since if the EN pin outputs 5V, then we want the buck to be off because this means either water is detected, and or the killswitch is enabled. LEDs are there so we can visually see if they are on or not.

5V to 3.3V LDO, and Sensors:

Here we just have a 3 pin header for LED strips which will be used for diagnosing issues, an IMU, and a pressure sensor. The LDO is essentially just used to power the pressure sensor (GY-MS5837-30BA). I also added 100nF decoupling capacitors for transients and zero ohm resistors for adjustability on all the data pins to the teensy.

7.4V to 5V 20A Buck Converter, Voltage and Current Measurement

We will likely not be pulling 20A, but we are powering a lot of parts. A Jetson Orin Nano (which to my understanding shouldn't be powered off 5V, minimum 7V, so I might unfortunately need to add an external step up converter, though some people said it worked on 5V so this is TBD), Raspberry pi 5 8gb, and a teensy, some sensors. We also will be powering servos on this rail with an external PWM controller. The output capacitors of the buck converter are really big, I also thought the sizing was incorrect but I rechecked calculations from the datasheet and it seemed right, I also was assuming the worst case, since the size of the caps is pretty large I added an 0603 bleed resistor. We also would like to find out the amount of current everything is pulling so I made a differential amplifier across a shunt resistor to find the current which we can calculate using I = V/R, potentially the resistor values will be adjusted so the op amp wouldnt be a unity gain amp and would be easier to read. The voltage of the battery is simply just with a voltage divider. All of this is using 0 ohm resistors so it can be adjusted later. We also are using XT30 connectors to connect to the Jetson, Pi, and anything else we would want to power, with small 0603 100nF decoupling capacitors for transients.

Teensy 4.1 corner, not a whole lot to say, just connects to headers.

Board Layout is still my area of least expertise and am really still trying to learn. So if you see anything that is questionable let me know. I also know that adding traces in the power plane (layer 3) is not ideal, but I think the tradeoff is worth it because of the oz copper I will be trying to use, and routing them on the front or back would make the GND plane to the XT30 connectors basically be "cut" which is bad because of the oz copper. I also added little astricts * so I can identify which pins are connected to something (like sensors), there is a PDF here that will show the pinnout. I also added a lot of test points and labeling. Also the RX and TX next to the teensy are so it can communicate with the pi and jetson orin nano.

Layers:
Front - Signals and GND Plane
2nd Layer - GND Plane
3rd Layer - Power Plane
4th Layer - Signals and GND Plane

Ideally, to cut cost, I am trying to use 1oz for the front and back, and 0.5 oz in the middle layers. Board size is 122mm x 71.5mm

