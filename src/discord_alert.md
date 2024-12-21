---

# **Arduino Sensor Data Discord Notification System**

This project is a Python script that reads **Arduino sensor data** and sends notifications to a **Discord webhook** based on specific conditions. When the **soil moisture value** falls below a threshold, a warning message is automatically sent.

---

## **1. Project Overview**

### **Features**
1. Reads sensor data via serial communication with Arduino.
2. Sends a notification to a Discord webhook when **soil moisture** is low.
3. Transmits all sensor data to Discord every 10 seconds.

---

## **2. Full Code**

```python
import serial  # For serial communication
import requests  # For HTTP requests
import time  # For delays

# Settings
WEBHOOK_URL = "please enter your webhook url"  # Enter your Discord webhook URL
SERIAL_PORT = "/dev/ttyUSB0"  # Check Arduino serial port (use `ls /dev/tty*` to verify)
BAUD_RATE = 9600  # Match the Baud Rate with your Arduino settings

# Send message to Discord
def send_to_discord(message):
    payload = {"content": message}  # Discord message format
    try:
        response = requests.post(WEBHOOK_URL, json=payload)
        if response.status_code == 204:
            print("Message successfully sent to Discord!")
        else:
            print(f"Failed to send: {response.status_code} - {response.text}")
    except Exception as e:
        print(f"An error occurred: {e}")

# Read data from Arduino
def read_sensor_data():
    try:
        # Start serial communication
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=2) as ser:
            print("Connecting to Arduino...")
            time.sleep(2)  # Wait to establish a stable connection
            while True:
                if ser.in_waiting > 0:
                    # Read data
                    data = ser.readline().decode('utf-8').strip()  # Read data from Arduino
                    print(f"Received sensor data: {data}")

                    # Parse data (e.g., "Soil Moisture:75, Light Intensity:28, Humidity:15, Temperature:24")
                    try:
                        values = {kv.split(":")[0].strip(): int(kv.split(":")[1].strip())
                                  for kv in data.split(",")}

                        # Check soil moisture condition
                        soil_moisture = values.get("Soil Moisture", 0)  # Use English keys from the sensor data
                        if soil_moisture < 30:
                            alert_message = f"⚠️ Soil moisture is low! Please water the plants! Current moisture level: {soil_moisture}"
                            print(alert_message)
                            send_to_discord(alert_message)

                        # Send all sensor data to Discord
                        send_to_discord(f"Sensor data alert: {data}")

                    except Exception as parse_error:
                        print(f"Data parsing error: {parse_error}")

                    time.sleep(10)  # Send data every 10 seconds

    except Exception as e:
        print(f"Serial communication error: {e}")

# Main function execution
if __name__ == "__main__":
    read_sensor_data()
```

---

## **3. Setup and Usage**

### **Environment Setup**
1. **Install Required Python Libraries**
   Install the necessary libraries:
```bash
pip install pyserial requests
```

2. **Set Discord Webhook URL**
   Generate a webhook URL from your Discord channel and set it in the `WEBHOOK_URL` variable.

3. **Check Arduino Serial Port**
   Identify the correct port for your system and set it in `SERIAL_PORT`:
   - **Linux/Mac**: `/dev/ttyUSB0` or `/dev/ttyACM0`
   - **Windows**: `COM3`, `COM4` etc.
     
4. **Arduino Baud Rate**
   Set the `BAUD_RATE` to match the baud rate in your Arduino code.

---

### **Execution**
```bash
python your_script_name.py
```

---

## **4. Key Features**

1. **Serial Communication**
   Reads sensor data from Arduino and decodes it using UTF-8.

2. **Data Parsing**
   Parses sensor data in the following format:
   ```
   "Soil Moisture: 250, Light Intensity: 500, Temperature: 25"
   ```

3. **Discord Notifications**
   - **Low Soil Moisture Condition**: Sends a warning message to Discord if `Soil Moisture < 30`.
   - **Periodic Data Transmission**: Sends all sensor data to Discord every 10 seconds.

---

## **5. Example Output**

### **Terminal Output**
```
Connecting to Arduino...
Received sensor data: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
⚠️ Soil moisture is low! Please water the plants! Current moisture level: 25
Message successfully sent to Discord!
Sensor data alert: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
```

### **Discord Message Example**
```
⚠️ Soil moisture is low! Please water the plants! Current moisture level: 25
Sensor data alert: Soil Moisture: 25, Light Intensity: 500, Temperature: 26
```

---

## **6. Notes**
1. **Discord Webhook Security**: Ensure the webhook URL is not exposed publicly.
2. **Port Configuration**: `The SERIAL_PORT` may vary depending on your system, so verify it carefully.
3. **Adjust Sensor Thresholds**: Modify condition values (e.g., `Soil Moisture < 30`) to match your sensors and environment.

---
