# 🌱  **Project Name: Smart Environmental Data Monitoring System**

## 📖 **Project Overview**
This project utilizes **Jetson Nano** and **Arduino sensors** to monitor and manage environmental data.
It collects real-time data on soil moisture, temperature, humidity, and light intensity, sending immediate **Discord notifications** to users in case of water scarcity. Furthermore, the system stores sensor data and provides users with easy access through **a Gradio UI-based chatbot**.

## 🚀 **Key Features**
- **Data Collection and Storage**:
  - S**oil Moisture Sensor**: Measures soil moisture levels in real-time
  - **DHT11 Sensor**: Collects temperature and humidity data in real-time
  - **Light Sensor**: Monitors ambient light intensity in real-time
  - All collected data is stored in **a CSV file**
    
- **Automated Notification System:**
  - Sends **Discord notifications** to users immediately when water scarcity is detected
  
- **Gradio UI-Based Chatbot**:
  - Offers an interactive **chatbot interface** for users to check sensor information
  - Visualizes data through Gradio UI, enhancing user accessibility
    
## 🛠️ **Technology Stack**
- **Programming Languages**: Python, C++
- **Platforms/Frameworks**: Jetson Nano, Arduino, Gradio
- **Hardware**:
  - Jetson Nano
  - Arduino
  - Sensors: Soil Moisture Sensor, DHT11 (Temperature and Humidity Sensor), Light Sensor
- **Additional Tools**: Discord API, CSV Data Storage


## 📁 **Project Structure**
```plaintext
📁 Smart Environmental Data Monitoring System/
├── README.md             # Project Description
├── src/                  # Source Code
│   ├── main_arduino.md   # Main Arduino sensor measurement code
│   ├── sensor_data.md    # Sensor data collection code
│   ├── discord_alert.md  # Discord notification system code
│   └── chatbot_ui.md     # Gradio UI-based chatbot code
├── failed_attempts/
│   └── gradio_graph_attempt.md  # Gradio UI graph attempt code
├── data/                 # CSV Data Storage Folder
│   └── sensor_data.csv   # Collected sensor data **sensor data 20241213.csv**
└── requirements.txt      # Dependency List
