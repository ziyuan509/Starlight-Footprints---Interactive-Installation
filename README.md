# Starlight Footprints - Interactive Installation

This project is an interactive installation that captures thermal footprints and transforms them into a dynamic, animated visual art piece. It uses an ESP32 with a thermal camera to detect heat signatures, which are then processed in Python and broadcasted as an NDI video stream for use in creative coding environments like TouchDesigner.

## Project Components

This repository contains the following key components:

*   **`MLX90640_Thermal_Camera/`**: Arduino code for the ESP32-S3 microcontroller. It reads data from the MLX90640 thermal sensor and broadcasts it over the network via UDP.
*   **`thermal_tracker_osc.py`**: A Python script that receives the raw thermal data, processes it to find the "hottest" point, and sends the coordinates as OSC messages. This is useful for simple tracking applications.
*   **`esp32_animator_bridge.py`**: The main Python application. It receives thermal data, manages a state machine for recording and playing back "footprint" trajectories, and renders the animation as an NDI stream.
*   **`requirements.txt`**: A list of the required Python libraries for this project.

## Features

*   **Real-time Thermal Tracking**: Captures a 32x24 thermal image to identify and track heat sources.
*   **Stateful Animation**: A state machine that waits, records a path of movement, and then plays back an animation of that path.
*   **NDI Video Streaming**: Broadcasts the final animation as a high-quality, low-latency NDI video stream, which can be easily used as a source in VJ software, game engines, and other visual applications.
*   **OSC Integration**: Provides an alternative data output via OSC for applications that require simple coordinate data.
*   **Decoupled Architecture**: The sensor (ESP32), processing (Python), and rendering (NDI) are decoupled, allowing for flexibility and scalability.

## How It Works

1.  **Sensing**: The `MLX90640_Thermal_Camera.ino` sketch running on the ESP32-S3 reads temperature data from the MLX90640 sensor. It then broadcasts the entire 768-pixel thermal data array over Wi-Fi using UDP broadcast packets.

2.  **Processing**:
    *   The `esp32_animator_bridge.py` script listens for these UDP packets and reconstructs the full thermal image.
    *   It uses OpenCV to find the centroid of the hottest area (the "hotspot").
    *   A state machine controls the logic:
        *   **WAITING**: The system is idle, waiting for the next animation cycle.
        *   **RECORDING**: When triggered, it records the path of the hotspot for a few seconds, storing a series of coordinates.
        *   **PLAYING**: It then plays back an animation, interpolating between the recorded points to create a smooth path.

3.  **Rendering & Output**:
    *   The animation is drawn onto a black background using OpenCV.
    *   This visual is then sent out as an NDI video stream named "ESP32_Animator_NDI".
    *   Simultaneously, the `thermal_tracker_osc.py` script can be run to send the hotspot's coordinates to an OSC listener.

## Hardware Requirements

*   **ESP32-S3 Microcontroller**: A powerful microcontroller with Wi-Fi capabilities.
*   **MLX90640 Thermal Camera**: A 32x24 pixel thermal imaging sensor.
*   **Computer**: A computer to run the Python scripts.

## Software Requirements

*   **Arduino IDE** or **PlatformIO**: To flash the firmware onto the ESP32.
*   **Python 3.x**
*   **Required Python Libraries**: Install them using the `requirements.txt` file:
    ```bash
    pip install -r requirements.txt
    ```
*   **NDI SDK**: The NDI Python library requires the NDI SDK to be installed on your system. You can download it from the [NDI website](https://ndi.tv/sdk/).

## Setup and Usage

1.  **ESP32 Setup**:
    *   Open the `MLX90640_Thermal_Camera/MLX90640_Thermal_Camera.ino` file in the Arduino IDE.
    *   Install the required libraries: `Adafruit_MLX90640`, `WiFi`, and `WiFiUdp`.
    *   Update the `WIFI_SSID` and `WIFI_PASS` variables with your Wi-Fi credentials.
    *   Make sure the correct GPIO pins for I2C (SDA and SCL) are set for your ESP32 board in the `Wire.begin()` function.
    *   Upload the sketch to your ESP32-S3.
    *   Open the Serial Monitor to see the IP address and confirm it's sending data.

2.  **Python Backend**:
    *   Make sure your computer is on the same Wi-Fi network as the ESP32.
    *   Install the required Python libraries using `pip install -r requirements.txt`.
    *   Run the main animator bridge:
        ```bash
        python esp32_animator_bridge.py
        ```
    *   You should see messages in the console indicating that the UDP server has started and the NDI sender has been created.

3.  **Receiving the NDI Stream**:
    *   Open your NDI-compatible software (e.g., TouchDesigner, Resolume, OBS with an NDI plugin).
    *   Add a new NDI source. You should see "ESP32_Animator_NDI" in the list of available sources.
    *   The animation of the thermal footprints will now be visible in your software.

4.  **(Optional) Using OSC**:
    *   If you only need the coordinates, you can run the OSC tracker instead:
        ```bash
        python thermal_tracker_osc.py
        ```
    *   Configure your receiving application to listen for OSC messages on port `9000` at the address `/possum/position`.

## Customization

*   **Animation Parameters**: In `esp32_animator_bridge.py`, you can adjust variables like `wait_duration`, `record_duration`, and `playback_duration` to change the timing of the animation.
*   **NDI Stream**: Change the `stream_name` to give your NDI stream a different name.
*   **OSC Address**: In `thermal_tracker_osc.py`, you can change the `TOUCHDESIGNER_IP` and `TOUCHDESIGNER_PORT` to send OSC data to a different application or machine.
*   **Thermal Thresholding**: The `ambient_temp` in `esp32_animator_bridge.py` and the threshold value in `thermal_tracker_osc.py` can be adjusted to change the sensitivity of the hotspot detection.
