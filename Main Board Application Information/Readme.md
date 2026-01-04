## Main Board Application Info  

### Voltage and Current Measurement

To find the voltage of the primary (electronics) battery, do the following math:

'Vbattery = Vmeasured * (10000 + 1000)/1000'

Note that Vmeasured is the voltage measured by the ADC. This is pin A12.

To find the current of the battery do the following math:

'Ibattery = Vmeasured / (Rsense * Gain)'  

Note that gain is 20, and Rsense is 0.005 ohms. This is pin A13.
