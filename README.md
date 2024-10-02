# Members
Caio S. F. da Silva: RM 554763   
Matheus R. Montovaneli: RM 555499
Lucas Vasquez Silva: RM 555159
Guilherme L. F. R. Gozzi: RM 555768
André Nakamatsu Rocha: RM 555004 


# ESP32 Sensor Data with MQTT

This project utilizes an **ESP32** microcontroller to collect data from two sensors: a **DHT22** (for temperature and humidity) and an **MQ-135** (for air quality). The sensor readings are published to an MQTT broker for remote monitoring.
## Link to the project
https://wokwi.com/projects/410030825523266561
## Components

- **ESP32**: The microcontroller used to run the code.
- **DHT22**: A sensor for measuring temperature and humidity.
- **MQ-135/Potentiometer (because wokwi doesn't have the MQ-135)**: A sensor for measuring air quality (CO2, smoke, etc.).
- **MQTT Broker**: The server that receives the sensor data.

## Libraries Used

- **WiFi.h**: Manages the ESP32's Wi-Fi connection.
- **PubSubClient.h**: Handles the MQTT communication.
- **DHTesp.h**: Communicates with the DHT22 sensor.

## How It Works

1. **Wi-Fi Connection**: The ESP32 connects to a Wi-Fi network.
2. **MQTT Connection**: Once Wi-Fi is connected, the ESP32 connects to the MQTT broker.
3. **Sensor Data Collection**:
   - The **DHT22** sensor collects temperature and humidity data.
   - The **MQ-135** sensor collects air quality data.
4. **Publishing Data**: The data from the sensors is sent to predefined MQTT topics on the broker:
   - Temperature is published to `Azure/Temperature`.
   - Humidity is published to `Azure/Humidity`.
   - Air quality is published to `Azure/AirQuality`.

## Getting Started

### Requirements

- **ESP32** development board.
- **Arduino IDE** with the ESP32 board package installed.
- The following libraries must be installed in the Arduino IDE:
  - `WiFi`
  - `PubSubClient`
  - `DHTesp`

### Hardware Setup

- **DHT22** sensor connected to pin **23** of the ESP32.
- **MQ-135** sensor connected to pin **34** of the ESP32.

### Code Configuration

1. **Wi-Fi Setup**: Replace the Wi-Fi SSID and password in the code with your network credentials.
   ```cpp
   const char* ssid = "your-SSID";
   const char* password = "your-PASSWORD";
   ```

2. **MQTT Broker**: Replace the MQTT broker IP with your server's IP or domain.
   ```cpp
   const char* mqtt_server = "your-mqtt-server-ip";
   ```

3. **Upload the Code**: Once the Wi-Fi and MQTT settings are configured, upload the code to your ESP32 using the Arduino IDE.

## How to Run

1. **Power the ESP32**: Provide power via USB or another source.
2. **Connect to Wi-Fi**: The ESP32 will attempt to connect to the configured Wi-Fi network.
3. **Connect to MQTT**: Once connected to Wi-Fi, the ESP32 will try to connect to the MQTT broker.
4. **Start Publishing**: Sensor data will be published to the MQTT topics every second.

### MQTT Topics

- **Temperature**: `Azure/Temperature`
- **Humidity**: `Azure/Humidity`
- **Air Quality**: `Azure/AirQuality`

### Example MQTT Payloads

- **Temperature**: `"25.4"`
- **Humidity**: `"60.2"`
- **Air Quality**: `"85"` (value mapped between 0-200)

## Code Breakdown

### Wi-Fi Connection

The `setup_wifi()` function initializes the Wi-Fi connection:
```cpp
void setup_wifi() {
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
}
```

### MQTT Reconnection

The `reconnect()` function attempts to reconnect to the MQTT broker if the connection is lost:
```cpp
void reconnect() {
  while (!client.connected()) {
    if (client.connect("ESP32_Client")) {
      // Connected to MQTT
    } else {
      delay(5000);  // Wait before retrying
    }
  }
}
```

### Sensor Data Collection

- **DHT22**: The function `DHT_dados()` collects temperature and humidity data and publishes it to MQTT topics.
  ```cpp
  void DHT_dados() {
    TempAndHumidity data = dhtSensor.getTempAndHumidity();
    client.publish("Azure/Temperature", String(data.temperature).c_str());
    client.publish("Azure/Humidity", String(data.humidity).c_str());
  }
  ```

- **MQ-135**: The function `AirQuality_dados()` reads air quality data and publishes it to the MQTT topic.
  ```cpp
  void AirQuality_dados() {
    int airQuality = analogRead(MQ135_PIN);
    int mappedAirQuality = map(airQuality, 0, 4095, 0, 200);
    client.publish("Azure/AirQuality", String(mappedAirQuality).c_str());
  }
  ```

### Main Loop

The `loop()` function ensures the ESP32 remains connected to MQTT and continuously publishes sensor data:
```cpp
void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
  DHT_dados();
  AirQuality_dados();
}
```

## License

This project is open-source and can be freely modified. Attribution is appreciated.

## Troubleshooting

- **Wi-Fi not connecting?** Ensure your Wi-Fi credentials (SSID and password) are correct.
- **MQTT broker connection issues?** Verify the broker address, network access, and that the broker is running.
- **Incorrect sensor readings?** Check the wiring and connections to the DHT22 and MQ-135 sensors.
## Node-RED Integration

This project requires **Node-RED** to handle, visualize, and process the sensor data received from the ESP32 via MQTT.

### What is Node-RED?

**Node-RED** is a flow-based development tool for visual programming, commonly used for wiring together hardware devices, APIs, and online services. In this project, it will act as the interface for receiving data from the ESP32 and displaying or processing the sensor readings.

### Setting Up Node-RED

1. **Install Node-RED**: You can install Node-RED by following the official instructions [here](https://nodered.org/docs/getting-started/).

2. **MQTT Integration in Node-RED**:
   - Once Node-RED is running, you can install the **MQTT** nodes to subscribe to the topics where your ESP32 is publishing data.
   - In the Node-RED interface, use the **MQTT in** node to subscribe to the `Azure/Temperature`, `Azure/Humidity`, and `Azure/AirQuality` topics.

