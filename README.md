# CSUS Competitive Robotics Club AUV PCB Descriptions

CSUS Competitive Robotics Club AUV PCB. This only includes the pinnouts and descriptions of the boards. 

Anything written below is not helpful unless you are looking for electrical system specific information. 

Note that the Main board and Secondary board have isolated grounds. Any interaction between them is using isolated components. They communicate with 5 pin JST connectors. Main board operates with a 2S lipo battery, while the secondary thruster PCB operates with a 4S battery. We isolated them because of voltage sag and emi. The hydrophone PCB will have the same ground has the 2S PCB, and will be powered with 3.3V. The images may NOT be the same as what is actually in the design files, however the differences will be minor, like silkscreen, pin placements, etc. Any major improvements will have updated pictures. 

MAIN BOARD SCHEMATIC AND BOARD LAYOUT INFORMATION: 

(Ideally open the schematic and board layout PDF while reading the information below, our logo will be added as well but this is purely cosmetic)

First I just want to say that the reason there is B_EN (buck converter enable) is that for our RoboSub competition we require a on off killswitch, so the EN pin will feed to the secondary board that will be connected to a P channel mosfet gate driver and the logic needed to be reversed for the buck converter, optionally we dont need to turn off our electronics however we decided to make the killswitch turn everything off, I added three options with 0 ohm resistors so we can adjust this later, we can either use B_EN, or EN, or just 5V so we can decide later too. Notibly, the 5V for the killswitch/water detection is different from the 5V for the other electronics, this is because if you use the same 5V, then it will be in a switching state and will not be stable meaning it will not either be on or off. 

Ill start from the top left of the schematic and work my way to the bottom right.

Water Detection, Killswitch, and 7.4V to 5V LDO:

The top left we have a LDO, that converts 7.4 to 5V for the killswitch and water detection logic. The killswitch is just connected to an external switch. The water detection how it will work is it uses 2.54mm headers that are glued to a sponge. Since this is pretty noisy there are RC filters. When water goes into the sponge, the wires short and will cause the AND gate output (where the EN pin is) to output 5V, which is good for the P channel mosfet driver since Vgs = Vg - Vs and means the mosfet will be off. The logic is inverted to the buck converter since if the EN pin outputs 5V, then we want the buck to be off because this means either water is detected, and or the killswitch is enabled. LEDs are there so we can visually see if they are on or not.
<img width="1182" height="877" alt="image" src="https://github.com/user-attachments/assets/c9dab479-52be-484f-96bb-fea8b2a8525c" />


5V to 3.3V LDO, and Sensors:

Here we just have a 3 pin header for LED strips which will be used for diagnosing issues, an IMU, and a pressure sensor. The LDO is essentially just used to power the pressure sensor (GY-MS5837-30BA). I also added 100nF decoupling capacitors for transients and zero ohm resistors for adjustability on all the data pins to the teensy.
<img width="1625" height="925" alt="image" src="https://github.com/user-attachments/assets/02f17311-95c6-4cd7-9198-d34ac28dbb22" />

7.4V to 5V 20A Buck Converter, Voltage and Current Measurement:

We will likely not be pulling 20A, but we are powering a lot of parts. A Jetson Orin Nano (which to my understanding shouldn't be powered off 5V, minimum 7V, so I might unfortunately need to add an external step up converter, though some people said it worked on 5V so this is TBD), Raspberry pi 5 8gb, and a teensy, some sensors. We also will be powering servos on this rail with an external PWM controller. The output capacitors of the buck converter are really big, I also thought the sizing was incorrect but I rechecked calculations from the datasheet and it seemed right, I also was assuming the worst case, since the size of the caps is pretty large I added an 0603 bleed resistor. We also would like to find out the amount of current everything is pulling so I made a differential amplifier across a shunt resistor to find the current which we can calculate using I = V/R, potentially the resistor values will be adjusted so the op amp wouldnt be a unity gain amp and would be easier to read. The voltage of the battery is simply just with a voltage divider. All of this is using 0 ohm resistors so it can be adjusted later. We also are using XT30 connectors to connect to the Jetson, Pi, and anything else we would want to power, with small 0603 100nF decoupling capacitors for transients.
<img width="1580" height="592" alt="image" src="https://github.com/user-attachments/assets/3b9242c9-09b2-4edc-9867-ccedce9373f7" />

Teensy 4.1:

Not a whole lot to say, just connects to headers.(the indented pins are the ones being used)
<img width="1315" height="1415" alt="image" src="https://github.com/user-attachments/assets/99b6c813-79ca-4093-85d4-1a776ba2c52a" />

Board Layout is still my area of least expertise and am still trying to learn and improve. So if you see anything that is questionable let me know. I also know that adding traces in the power plane (layer 3) is not ideal, but I think the tradeoff is worth it because of the oz copper I will be trying to use, and routing them on the front or back would make the GND plane to the XT30 connectors basically be "cut" which is bad because of the oz copper. I also added little astricts * so I can identify which pins are connected to something (like sensors), there is a PDF here that will show the pinnout. I also added a lot of test points and labeling. Also the RX and TX next to the teensy are so it can communicate with the pi and jetson orin nano. I separated the ground loops, so the basically the buck converter has one loop back to the battery, and all the rest of the components share the same ground loop, of course while making sure there are no ground islands and that the board follows basic DRC. 

Layers:
Front (red) - Signals and GND Plane

2nd Layer (green) - GND Plane

3rd Layer (orange) - Power Plane

4th Layer (blue) - Signals and GND Plane

Ideally, to cut cost, I am trying to use 1oz for the front and back, and 0.5 oz in the middle layers. Board size is 122mm x 71.5mm

All Planes visible:
<img width="3202" height="1982" alt="image" src="https://github.com/user-attachments/assets/73ba9e6a-35f5-49b3-8e96-cd4bfb8ca251" />

Front Plane:
<img width="3195" height="1975" alt="image" src="https://github.com/user-attachments/assets/658d4544-61e5-4d7c-bdad-b76de8dd8b14" />

2nd Layer:
<img width="3197" height="1982" alt="image" src="https://github.com/user-attachments/assets/ac8e664b-e581-4a77-bde6-31691627d380" />

3rd Layer:
<img width="3200" height="1977" alt="image" src="https://github.com/user-attachments/assets/fd968ce7-6fc8-4446-80cf-46d3da47ebca" />

4th Layer:
<img width="3200" height="1967" alt="image" src="https://github.com/user-attachments/assets/146369e8-8507-47c1-ba52-a9137b95a1d3" />

3D View (Front):
<img width="3355" height="2087" alt="image" src="https://github.com/user-attachments/assets/6efef9bb-d6c4-425c-8335-7d4276d7130a" />

3D View (Back):
<img width="3355" height="2085" alt="image" src="https://github.com/user-attachments/assets/e175ebd8-d029-499f-97ca-cc247c428fa7" />
