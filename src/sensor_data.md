---

## Collecting Arduino Sensor Data (collect_sensor_data.md)

This script provides **a Python program** using **Pandas** to collect Arduino sensor data in real time and save it into a CSV file. 
This code is executed in **a virtual environment** on **the Jetson Nano** using **Jupyter Notebook**.

---

### **Full Code**

```python
import serial
import time
import pandas as pd

# Serial communication setup
port = "/dev/ttyUSB0"  # Serial port for Arduino
baudrate = 9600        # Baud rate matching Arduino
arduino = serial.Serial(port, baudrate, timeout=1)

# List for storing data
data = []

# Initialize current date
current_date = time.strftime("%Y-%m-%d")

try:
    print("Collecting data from Arduino... Press Ctrl+C to stop.")
    while True:
        if arduino.in_waiting > 0:  # Check if data is available on the serial port
            line = arduino.readline().decode("utf-8").strip()  # Read and decode data
            
            # Data filtering: Process only sensor data
            if "Soil Moisture:" in line and "Light Intensity:" in line:
                timestamp = time.strftime("%Y-%m-%d %H:%M:%S")  # Save the current timestamp
                
                # Parse sensor data
                parts = line.split(", ")
                soil_moisture = parts[0].split(":")[1].strip()  # Soil Moisture value
                light_intensity = parts[1].split(":")[1].strip()  # Light Intensity value
                humidity = parts[2].split(":")[1].strip()  # Humidity value
                temperature = parts[3].split(":")[1].strip()  # Temperature value
                
                print(f"{timestamp}, Soil Moisture: {soil_moisture}, Light Intensity: {light_intensity}, Humidity: {humidity}, Temperature: {temperature}")
                
                # Store data
                data.append({
                    "Timestamp": timestamp,
                    "Soil Moisture (%)": soil_moisture,
                    "Light Intensity (%)": light_intensity,
                    "Humidity (%)": humidity,
                    "Temperature (C)": temperature
                })

                # Create DataFrame
                df = pd.DataFrame(data)

                # Generate file name (based on current date)
                file_name = f"sensor_data_{current_date.replace('-', '')}.csv"

                # Save DataFrame to CSV file
                df.to_csv(file_name, index=False, encoding="utf-8")
                print(f"Data saved to {file_name}")

except KeyboardInterrupt:
    print("Data collection stopped.")

finally:
    # Save remaining data
    if data:
        df = pd.DataFrame(data)
        file_name = f"sensor_data_{current_date.replace('-', '')}.csv"
        df.to_csv(file_name, index=False, encoding="utf-8")
        print(f"Final data saved to {file_name}")

    # Close the port
    arduino.close()
```

---

### **Project Overview**

This script performs the following tasks:

1. **Arduino Serial Communication Setup**: Connects to the serial port to read data.
2. **Data Filtering and Parsing**: Reads sensor data (soil moisture, light intensity, temperature, and humidity) sent by Arduino and parses it correctly.
3. **Real-Time Data Saving**: Saves collected data to a CSV file in real time.
4. **Data Preservation on Exit**: Safely saves any remaining data when the program is terminated.

---

### **Usage**

1. Upload and run the Arduino code (`mainarduino.md`) on your board.
2. Execute the Python script:
   ```bash
   python collect_sensor_data.py
   ```
3. Press `Ctrl+C` to stop the program.
4. Collected data will be saved to a file named **`sensor_data_YYYYMMDD.csv`**.

---

### **Example Output (Terminal)**

```
2024-06-01 14:23:45, Soil Moisture: 45, Light Intensity: 78, Humidity: 60, Temperature: 24
Data saved to sensor_data_20240601.csv
2024-06-01 14:23:47, Soil Moisture: 43, Light Intensity: 80, Humidity: 59, Temperature: 23
Data saved to sensor_data_20240601.csv
```

---

### **Example Output File (CSV)**

| Timestamp           | Soil Moisture (%) | Light Intensity (%) | Humidity (%) | Temperature (C) |
|---------------------|-------------------|---------------------|-------------|-----------------|
| 2024-06-01 14:23:45 | 45                | 78                  | 60          | 24              |
| 2024-06-01 14:23:47 | 43                | 80                  | 59          | 23              |

---

### **Notes**

1. **Serial Port Configuration**: Modify `/dev/ttyUSB0` (Linux/Mac) or `COM3` (Windows) according to your system.
2. **Baud Rate**: Ensure that the baud rate in the Arduino code matches the baudrate in this script (`9600 by default`).
3.** Data Preservation on Exit**: When stopping the program with `Ctrl+C`, all collected data will be saved automatically.

---
