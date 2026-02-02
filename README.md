# CSUS Competitive Robotics Club AUV PCB Descriptions

CSUS Competitive Robotics Club AUV PCB. This only includes the pinnouts and descriptions of the boards. 

## Main Board Technical Information 
Note that anything written below is not helpful unless you are looking for electrical system specific information. If you are looking for how to use this board, go to the `"Main Board Application Information"` folder. 

Note that the Main board and Secondary board have SHARED grounds, they will be connected using the mounting holes and a spade connector. The main board mainly powers all of the electronics, while the secondary board will mainly power the thrusters. They communicate with 5 pin JST connectors. Main board operates with a 2S lipo battery, while the secondary thruster PCB operates with a 4S battery. The hydrophone PCB will have the same ground as the 2S PCB, and will be powered with 5V. The images may NOT be the same as what is actually in the design files, however the differences will be minor, like silkscreen, placements, etc. Any major improvements will have updated pictures. Something else to note is that there is a chance in the future we will switch to a 12V tether instead, so this is designed for an input voltage of 7.4-12V, although this board was mainly designed with the intent of a 2S battery. 

## Main Board Schematic and Board Layout Information

(Ideally if you can, open the schematic and board layout PDF while reading the information below)

First I just want to say that the reason there is B_EN (buck converter enable) is that for our RoboSub competition we are required to have a on off killswitch. So the B_EN will specifically be for the buck converters simply because of how our killswitch logic works for our secondary board, which turns off the thrusters. The EN pin will feed to the secondary board that will be connected to a N channel MOSFET, that helps drive a P channel MOSFET and the logic needed to be reversed for the buck converter, optionally we dont need to turn off our electronics however we decided to make the killswitch turn everything off, I added three selectable options with 0 ohm resistors so we can adjust this later, so we can either use B_EN, or EN, or just 5V to all of the buck converters so we can decide later too. Notibly, the 5V for the killswitch/water detection circuitry is different from the 5V for the other electronics, this is because if you try to use your regulated 5V from the buck converter that powers like the Pi, then it will be in a switching state and will not be stable meaning it will not either be on or off becauses it will be trying to power itself on/off.

Ill start from the top left of the schematic and work my way to the bottom right.

## Water Detection, Killswitch, and 7.4V to 5V LDO

The top left we have a LDO, that converts 7.4 to 5V for the killswitch and water detection logic. The killswitch is just connected to an external switch. The water detection how it will work is it uses 2.54mm headers that are glued to a sponge. Since this is pretty noisy there are RC filters. When water goes into the sponge, the wires short and will cause the AND gate output (where the EN pin is) to output 5V, which is good for driving the P channel MOSFET, hence why EN is connected to the secondary board, which also means the MOSFET will be off. The logic is inverted to the buck converter since if the EN pin outputs 5V, then we want the buck to be off because this means either water is detected, and or the killswitch is enabled. LEDs are there so we can visually see if they are on or not.
<img width="1545" height="1152" alt="image" src="https://github.com/user-attachments/assets/a69cf71d-9105-4e98-a1bc-6b39f59cb9e0" />


## 5V to 3.3V LDO, and Sensors

Here we just have a 3 pin header for LED strips which will be used for diagnosing issues, an IMU, and a pressure sensor. The LDO is essentially just used to power the pressure sensor (GY-MS5837-30BA). I also added 100nF decoupling capacitors for transients and zero ohm resistors for adjustability on all the data pins to the teensy.
<img width="1440" height="815" alt="image" src="https://github.com/user-attachments/assets/44ff4c8d-ecff-471a-addc-76ebbbf9dd7d" />

## 7.4V to 5V 20A Buck Converter, Voltage and Current Measurement

We will likely not be pulling 20A, but we are powering a lot of parts. A Raspberry pi 5 8gb, and a teensy, some sensors. We also will be powering servos on this rail with an external PWM controller. The output capacitors of the buck converter are really big, I also thought the sizing was incorrect but I rechecked calculations from the datasheet and it seems right, I also was assuming the worst case. Since the size of the caps is pretty large I added an 0603 bleed resistor. The voltage of the battery is simply measured just with a voltage divider. The current measurement uses an IC, which with the formula in the application notes, at 20A we should get 2V to the ADC of the teensy which is safe. All of this is using 0 ohm resistors so it can be adjusted later. We also are using XT30 connectors to connect to the Jetson, Pi, and anything else we would want to power, with small 0603 100nF decoupling capacitors for transients. 
<img width="1375" height="510" alt="image" src="https://github.com/user-attachments/assets/2d405ae0-6068-4d61-822f-7ec453c4c2ab" />

## Teensy 4.1

Not a whole lot to say, it just connects to 2.54mm pin headers (the indented pins are the ones being used by sensors)
<img width="1130" height="1227" alt="image" src="https://github.com/user-attachments/assets/607d1fd8-d2e3-4d8e-8c7a-1c42515b3caa" />

Board Layout is still my area of least expertise and am still trying to learn and improve. I know that adding traces in the power plane (layer 3) is not ideal, but I think the tradeoff is worth it because of the oz copper I will be trying to use (which is relatively low), and routing them on the front or back would make the GND plane to the XT30 connectors basically be "cut" which is bad because of the oz copper and the trace wouldn't carry as much current. I also added little astricts * next to the pin headers for the teensy so I can identify which pins are connected to something (like sensors), there is a PDF here that will show the pinnout. I also added some test points and labeling. Also the RX and TX next to the teensy are so it can communicate with the pi and jetson orin nano. I separated the ground loops, so the basically the buck converter has only a few possible loops back to the battery, and all the rest of the components share a differnet ground loop all connecting back to the battery, of course while making sure there are no ground islands and that the board follows basic DRC. The buck converter has many vias, to make the return path to the battery low impedance. Because the path of the buck converter is slightly divided with some planes being smaller, meaning more resistance, I added more vias onto the larger pad to ensure more power goes onto that larger plane. 

## Layers
Front (red) - Signals and GND Plane

2nd Layer (green) - GND Plane

3rd Layer (orange) - Power Plane

4th Layer (blue) - Signals and GND Plane

Ideally, to cut cost, I am trying to use 1oz for the front and back, and 0.5 oz in the middle layers. 

Board size is 122mm x 71.5mm, radius fillet on the edges are 2mm, and uses M2 screws for mounting holes. 

Previous mistake fixes/revisions that this board has:
* Teensy pinnout fixed, made pins 0 and 1 (UART pins) close together, also I flipped the labels for the teensy pinout and also added a 5V output on some pin headers

Minor issues in the layout that you WILL likely see in the following images that since then have been fixed/improved:
*  4 pin JST connector instead of 3 for the secondary board, added a buck converter enable pin

All Planes visible:

Front Plane:
<img width="3197" height="1987" alt="image" src="https://github.com/user-attachments/assets/5c4c8e2a-c100-4fe3-87f0-f1689ddce3ab" />

2nd Layer:
<img width="3205" height="1982" alt="image" src="https://github.com/user-attachments/assets/f1a1762f-8bf2-408d-ba95-140f0de7a111" />

3rd Layer (again, even though not optimal, a lot of traces are here because if I put them on the top or bottom layer, the ground or power plane will be "cut" and those planes have a higher oz copper meaning they can carry more current):
<img width="3195" height="1975" alt="image" src="https://github.com/user-attachments/assets/d71142be-e8f3-4fd3-9519-96510bc1229a" />

4th Layer:
<img width="3200" height="1980" alt="image" src="https://github.com/user-attachments/assets/015b9145-f4ac-480c-9beb-b8c5d8f568c6" />

3D View (Front):
<img width="3357" height="2152" alt="image" src="https://github.com/user-attachments/assets/a1d3d56c-5bf5-4e2b-9cd9-c05cde1f6594" />

3D View (Back):
<img width="3350" height="2075" alt="image" src="https://github.com/user-attachments/assets/db63ad5c-2139-4ad1-ae18-d8620f745cc3" />
