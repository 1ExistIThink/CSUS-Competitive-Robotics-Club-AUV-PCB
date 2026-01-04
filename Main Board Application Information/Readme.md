# Main Board Application Info

To measure the voltage and current of the primary electronics battery:

**Voltage Measurement:**

Vbattery = Vmeasured * (10000 + 1000) / 1000  
- `Vmeasured` = ADC pin A12  
- Example: if Vmeasured = 2.5 V → Vbattery = 2.5 * 11 = 27.5 V  

**Current Measurement:**

Ibattery = Vmeasured / (Rsense * Gain)  
- `Vmeasured` = ADC pin A13  
- Rsense = 0.005 Ω  
- Gain = 20  
- Example: if Vmeasured = 4 V → Ibattery = 4 / (0.005 * 20) = 40 A  

**Notes:**  
- Make sure ADC pins match hardware: A12 = voltage, A13 = current  
- Rsense and Gain must match INA180 settings  
- Adjust gain or Rsense for different current ranges  

**Suggested GitHub folder structure:**

