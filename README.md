# Ex-12 Mini Project
# ADVANCED SATELLITE FOR TELEMETRY AND REMOTE ACCESS

## 1. TOPIC

Design and implementation of an ESP32-based satellite telemetry system using MPU6050 and NEO-6M GPS with wireless data transmission and web-based remote monitoring.

## 2. AIM

To design and implement a compact embedded telemetry system using ESP32 that collects real-time location, motion, and orientation data from GPS and MPU6050 sensors, transmits the data through Wi-Fi, and displays the telemetry information on a web-based interface.

## 3. REQUIREMENTS

### Hardware Requirements

* ESP32 Development Board
* MPU6050 Accelerometer and Gyroscope
* NEO-6M GPS Module
* Battery
* Battery Management System (BMS)
* 5 V Buck Converter
* 3.3 V Buck Converter
* Jumper Wires
* Breadboard
* USB Cable
* Computer or Mobile Device

### Software Requirements

* Arduino IDE
* ESP32 Board Package
* MPU6050 Library
* TinyGPS++ Library
* Wi-Fi Library
* Web Browser

## 4. PROCEDURE

1. Connect the MPU6050 sensor to the ESP32 using the I2C interface.
2. Connect the NEO-6M GPS module to the ESP32 using the UART interface.
3. Provide regulated power to the ESP32, MPU6050, and GPS module.
4. Install the required ESP32 board package and libraries in Arduino IDE.
5. Write and upload the telemetry program to the ESP32.
6. Initialize the MPU6050 and GPS modules.
7. Connect the ESP32 to a Wi-Fi network.
8. Read acceleration and gyroscope values from the MPU6050.
9. Read latitude and longitude information from the NEO-6M GPS.
10. Process and format the sensor data.
11. Create a web server on the ESP32.
12. Display the collected telemetry data on the web interface.
13. Access the web interface using a computer or mobile device connected to the same network.
14. Continuously update and monitor the telemetry information.

## 5. PROGRAM

```cpp
#include <WiFi.h>
#include <Wire.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>
#include <TinyGPS++.h>
#include <HardwareSerial.h>

// Wi-Fi credentials
const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

// Create objects
Adafruit_MPU6050 mpu;
TinyGPSPlus gps;
HardwareSerial GPS_Serial(1);

WiFiServer server(80);

// GPS pins
#define GPS_RX 16
#define GPS_TX 17

void setup()
{
    Serial.begin(115200);

    // Initialize I2C
    Wire.begin();

    // Initialize MPU6050
    if (!mpu.begin())
    {
        Serial.println("MPU6050 not detected!");
        while (1)
        {
            delay(10);
        }
    }

    Serial.println("MPU6050 initialized.");

    // Initialize GPS
    GPS_Serial.begin(9600, SERIAL_8N1, GPS_RX, GPS_TX);

    // Connect to Wi-Fi
    WiFi.begin(ssid, password);

    Serial.print("Connecting to Wi-Fi");

    while (WiFi.status() != WL_CONNECTED)
    {
        delay(500);
        Serial.print(".");
    }

    Serial.println();
    Serial.println("Wi-Fi Connected!");
    Serial.print("IP Address: ");
    Serial.println(WiFi.localIP());

    // Start web server
    server.begin();
}

void loop()
{
    // Read GPS data
    while (GPS_Serial.available() > 0)
    {
        gps.encode(GPS_Serial.read());
    }

    // Read MPU6050 data
    sensors_event_t acceleration;
    sensors_event_t gyro;
    sensors_event_t temperature;

    mpu.getEvent(&acceleration, &gyro, &temperature);

    // Store GPS data
    double latitude = 0.0;
    double longitude = 0.0;

    if (gps.location.isValid())
    {
        latitude = gps.location.lat();
        longitude = gps.location.lng();
    }

    // Print telemetry data
    Serial.println("----------------------------------");
    Serial.println("SATELLITE TELEMETRY DATA");

    Serial.print("Latitude: ");
    Serial.println(latitude, 6);

    Serial.print("Longitude: ");
    Serial.println(longitude, 6);

    Serial.print("Acceleration X: ");
    Serial.print(acceleration.acceleration.x);
    Serial.println(" m/s^2");

    Serial.print("Acceleration Y: ");
    Serial.print(acceleration.acceleration.y);
    Serial.println(" m/s^2");

    Serial.print("Acceleration Z: ");
    Serial.print(acceleration.acceleration.z);
    Serial.println(" m/s^2");

    Serial.print("Gyroscope X: ");
    Serial.print(gyro.gyro.x);
    Serial.println(" rad/s");

    Serial.print("Gyroscope Y: ");
    Serial.print(gyro.gyro.y);
    Serial.println(" rad/s");

    Serial.print("Gyroscope Z: ");
    Serial.print(gyro.gyro.z);
    Serial.println(" rad/s");

    // Check for web client
    WiFiClient client = server.available();

    if (client)
    {
        String request = client.readStringUntil('\r');
        client.flush();

        // HTTP response
        client.println("HTTP/1.1 200 OK");
        client.println("Content-Type: text/html");
        client.println("Connection: close");
        client.println();

        client.println("<!DOCTYPE html>");
        client.println("<html>");
        client.println("<head>");
        client.println("<meta http-equiv='refresh' content='2'>");
        client.println("<title>Satellite Telemetry</title>");

        client.println("<style>");
        client.println("body { font-family: Arial; text-align: center; }");
        client.println("table { margin: auto; border-collapse: collapse; }");
        client.println("th, td { border: 1px solid black; padding: 10px; }");
        client.println("h1 { margin-bottom: 20px; }");
        client.println("</style>");

        client.println("</head>");
        client.println("<body>");

        client.println("<h1>Advanced Satellite Telemetry</h1>");

        client.println("<table>");

        client.println("<tr>");
        client.println("<th>Parameter</th>");
        client.println("<th>Value</th>");
        client.println("</tr>");

        client.println("<tr>");
        client.println("<td>Latitude</td>");
        client.print("<td>");
        client.print(latitude, 6);
        client.println("</td>");
        client.println("</tr>");

        client.println("<tr>");
        client.println("<td>Longitude</td>");
        client.print("<td>");
        client.print(longitude, 6);
        client.println("</td>");
        client.println("</tr>");

        client.println("<tr>");
        client.println("<td>Acceleration X</td>");
        client.print("<td>");
        client.print(acceleration.acceleration.x);
        client.println(" m/s²</td>");
        client.println("</tr>");

        client.println("<tr>");
        client.println("<td>Acceleration Y</td>");
        client.print("<td>");
        client.print(acceleration.acceleration.y);
        client.println(" m/s²</td>");
        client.println("</tr>");

        client.println("<tr>");
        client.println("<td>Acceleration Z</td>");
        client.print("<td>");
        client.print(acceleration.acceleration.z);
        client.println(" m/s²</td>");
        client.println("</tr>");

        client.println("<tr>");
        client.println("<td>Gyroscope X</td>");
        client.print("<td>");
        client.print(gyro.gyro.x);
        client.println(" rad/s</td>");
        client.println("</tr>");

        client.println("<tr>");
        client.println("<td>Gyroscope Y</td>");
        client.print("<td>");
        client.print(gyro.gyro.y);
        client.println(" rad/s</td>");
        client.println("</tr>");

        client.println("<tr>");
        client.println("<td>Gyroscope Z</td>");
        client.print("<td>");
        client.print(gyro.gyro.z);
        client.println(" rad/s</td>");
        client.println("</tr>");

        client.println("</table>");

        client.println("<p>Telemetry updates automatically every 2 seconds.</p>");

        client.println("</body>");
        client.println("</html>");

        delay(1);
        client.stop();
    }

    delay(2000);
}
```

## 6. OUTPUT

### Serial Monitor Output

```text
Wi-Fi Connected!
IP Address: 192.168.1.105

----------------------------------
SATELLITE TELEMETRY DATA
Latitude: 13.082680
Longitude: 80.270718

Acceleration X: 0.15 m/s^2
Acceleration Y: 0.32 m/s^2
Acceleration Z: 9.71 m/s^2

Gyroscope X: 0.02 rad/s
Gyroscope Y: 0.01 rad/s
Gyroscope Z: 0.03 rad/s
```

### Web Interface Output

```text
-------------------------------------
       ADVANCED SATELLITE TELEMETRY
-------------------------------------

Latitude          : 13.082680
Longitude         : 80.270718

Acceleration X    : 0.15 m/s²
Acceleration Y    : 0.32 m/s²
Acceleration Z    : 9.71 m/s²

Gyroscope X       : 0.02 rad/s
Gyroscope Y       : 0.01 rad/s
Gyroscope Z       : 0.03 rad/s

Telemetry updates automatically
every 2 seconds.
```

## 7. RESULT

The ESP32-based satellite telemetry system was successfully designed and implemented. The system acquired real-time location data using the NEO-6M GPS and motion and orientation data using the MPU6050 sensor. The ESP32 processed the sensor information and transmitted the telemetry data through Wi-Fi.

The collected information was successfully displayed through a web-based interface, demonstrating the basic telemetry lifecycle of sensing, processing, wireless transmission, and remote visualization.
