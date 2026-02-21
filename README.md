# CSUS Competitive Robotics Club AUV PCB Descriptions

CSUS Competitive Robotics Club AUV PCB. This only includes the pinnouts and descriptions of the boards. 

Note that anything written below is not helpful unless you are looking for electrical system specific information. If you are looking for how to use this board, go to the `"Application Information"` folders. 

Note that the Main board and Secondary board have shared grounds, they will be connected using the mounting holes and a spade connector. The main board mainly powers all of the electronics, while the secondary board will mainly power the thrusters. They "communicate" with 4 pin JST connectors. Main board operates with a 2S lipo battery, while the secondary thruster PCB operates with a 4S battery. The hydrophone PCB will have the same ground as the 2S PCB, and will be powered with 5V. The images may NOT be the same as what is actually in the design files, however the differences will be minor, like silkscreen, placements, etc. Any major improvements will have updated pictures.

## Electrical System information 
Before going into this, it is important to know how our system will communicate and use data. We are using a 4S battery connected to the Secondary ESC board and a 2S battery connected to the Main board. The grounds are connected together through spade connectors at the mounting holes. Communication between all SBCs, including the Jetson Orin Nano and Raspberry Pi, and the microcontroller, the Teensy 4.1, is handled over serial using UART. The Teensy 4.1 connects directly to all sensors, including the pressure sensor, hydrophones, diagnostic LED strip, and current and voltage sensing on both boards. By handling sensors and lower level tasks on the Teensy, the SBCs can focus on the more computationally intensive processes such as stereo vision and control without any significant delays. 

For the hydrophones in particular, the Teensy will preform sampling to capture the 25 to 40 kHz pinger signals. The sampling rate will be chosen well above the Nyquist minimum to allow accurate frequency detection. Each ADC on the Teensy has a sample rate of 1 Msps. Since the Teensy has two ADCs, this means you can hypothetically syncrinously sample two hydrophones at once. However, we are running many more sensors and three hydrophones, so the delay times between the hydrophones will not be as good as an FPGA, but will be usable. We plan on connecting the hydrophone signals to A0 and A1 which will use ADC1, and A17 which will use ADC2. From our research, A0–A9 will use ADC1 (by default, but reassignable to ADC2), A10–A11 will use ADC1, A12–A17 will use ADC2.

The theoretical minimum sampling rate required to capture a 40 kHz signal is 80 kSps, but at that limit it provides little tolerance for error. By sampling at a higher rate, multiple samples are obtained per waveform period, improving the resolution and time difference of arrival calculations between hydrophones. Because two hydrophones share ADC1, there will be a small sequential sampling delay between those channels, but this delay is predictable. Signals such as battery voltage and current measurements will be sampled at much lower rates because they are DC measurements and do not require much sampling. 

## Main Board Technical Information 
Something to note is that there is a chance in the future we will switch to a 12V tether instead, so this is designed for an input voltage of 7.4-12V, although this board was mainly designed with the intent of a 2S battery. 

## Main Board Schematic and Board Layout Information

(Ideally if you can, open the schematic and board layout PDF while reading the information below)

Firstly, you will see B_EN and EN, these are both used for killswitch logic. The reason B_EN, or buck converter enable, exists is because the RoboSub competition requires a physical on off killswitch. B_EN is dedicated specifically to controlling the buck converters due to how our killswitch logic operates on the secondary board which shuts off the thrusters. The EN pin feeds to the secondary board and connects to an N channel MOSFET that drives a P channel MOSFET, which requires the logic for the buck converter enable to be inverted. Although we are not strictly required to power down all electronics when the killswitch is enabled, we chose to design the system so that the killswitch can shut everything off.

To keep this flexible, I added three selectable options using 0 ohm resistors so we can later choose whether each buck converter is controlled by B_EN, EN, or tied directly to 5V, allowing us to decide what is turned off and how without redesigning the board. Notibly, the 5V rail for the killswitch and water detection circuitry is separate from the main regulated 5V rail that powers electronics such as the Raspberry Pi. If the killswitch circuitry were powered from the same buck converter output that it controls, the regulator could enter an unstable switching condition, since it would be trying to power itself on and off rather than reaching a stable state. 

I'll start from the top left of the schematic and work my way to the bottom right.

## Water Detection, Killswitch, and 7.4V to 5V LDO

In the top left of the schematic there is an LDO that steps 7.4V down to 5V for the killswitch and water detection logic. The killswitch itself connects to an externally accessible switch. The water detection system uses 2.54 mm headers that are glued into a sponge. Since this is relatively noisy, RC filters are added. When water gets into the sponge, the exposed header pins short together, which causes the AND gate output, connected to the EN pin, to go to 5V. This signal is used to drive a P channel MOSFET through an N channel MOSFET on the secondary board, which is why EN is connected there. In this particular condition the MOSFET turns off. The logic is intentionally inverted relative to the buck converters, because when the EN signal is at 5V it indicates either water detection or an active killswitch, and therefore the buck converters must be disabled. LEDs are there so we can visually see the state of our system.

<img width="1542" height="1140" alt="image" src="https://github.com/user-attachments/assets/2fc8909f-eaff-4e69-b7b8-8cd086723d26" />


## 5V to 3.3V LDO, and Sensors

Here we just have a 3 pin header for LED strips which will be used for diagnosing issues, an IMU, and a pressure sensor. The LDO is essentially just used to power the pressure sensor (GY-MS5837-30BA). I also added 100nF decoupling capacitors for transients and zero ohm resistors for adjustability on all the data pins to the teensy.
<img width="1417" height="805" alt="image" src="https://github.com/user-attachments/assets/ab34c76d-71c9-4bb2-a13c-35064d849c17" />

## 7.4V to 5V 20A Buck Converter, Voltage and Current Measurement

We will likely not be pulling 20A, but we are powering a lot of parts. A Raspberry pi 5 8gb, and a teensy, some sensors. We also will be powering servos on this rail with an external PWM controller. The output capacitors of the buck converter are really big, I also thought the sizing was incorrect but I rechecked calculations from the datasheet and it seems right, I also was assuming the worst case. Since the size of the caps is pretty large I added an 0603 bleed resistor. The voltage of the battery is simply measured just with a voltage divider. The current measurement uses an IC, which with the formula in the application notes, at 20A we should get 2V to the ADC of the teensy which is safe. All of this is using 0 ohm resistors so it can be adjusted later. We also are using XT30 connectors to connect to the Jetson, Pi, and anything else we would want to power, with small 0603 100nF decoupling capacitors for transients. 
<img width="1370" height="507" alt="image" src="https://github.com/user-attachments/assets/232e6358-cf3b-4fbc-bbc1-c6b05514c7fc" />

## Teensy 4.1

Not a whole lot to say, it just connects to 2.54mm pin headers (the indented pins are the ones being used by sensors)
<img width="1130" height="1227" alt="image" src="https://github.com/user-attachments/assets/de5ef8cd-3ddf-499f-b850-7f93b6b028e4" />

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
* Significantly different will update later, added a XT30 directly to the battery through the fuse for servos 2/14/2026
Minor issues in the layout that you WILL likely see in the following images that since then have been fixed/improved:
*  4 pin JST connector instead of 3 for the secondary board, added a buck converter enable pin

All Planes visible:
<img width="2867" height="1790" alt="image" src="https://github.com/user-attachments/assets/3872f670-e304-46e8-9c6f-23365f9067aa" />

Front Plane:
<img width="2840" height="1762" alt="image" src="https://github.com/user-attachments/assets/ab28dff3-862c-4dbe-a76d-1149350eac8c" />

2nd Layer:
<img width="2857" height="1742" alt="image" src="https://github.com/user-attachments/assets/30d60f44-be73-4e82-a5a9-44a3a1fb2a26" />

3rd Layer (again, even though not optimal, a lot of traces are here because if I put them on the top or bottom layer, the ground or power plane will be "cut" and those planes have a higher oz copper meaning they can carry more current):
<img width="2872" height="1762" alt="image" src="https://github.com/user-attachments/assets/1f12dddc-8846-4c0d-848e-8465923c5291" />

4th Layer:
<img width="2860" height="1735" alt="image" src="https://github.com/user-attachments/assets/e79f5df1-c4c6-4898-ae06-d55da5759f15" />

3D View (Front):
<img width="3020" height="2060" alt="image" src="https://github.com/user-attachments/assets/ed8082e4-8651-453e-b476-1713b6101fcd" />

3D View (Back):
<img width="3250" height="2090" alt="image" src="https://github.com/user-attachments/assets/56a42693-6507-41bb-897a-f0e3bcc9ebbb" />



## Secondary Board Technical Information 
Note that anything written below is not helpful unless you are looking for electrical system specific information. If you are looking for how to use this board, go to the `"Application Information"` folders. 

Something else to note is that there is a chance in the future we will switch to a 12V tether instead, so this is designed for an input voltage of 12-16.8V, although this board was mainly designed with the intent of a 4S battery. 

## Secondary Board Schematic and Board Layout Information
