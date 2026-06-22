# IoT Gas Monitoring Station with EMQX Cloud Integration

## 📌 Project Overview
This project focuses on building a real-world IoT telemetry pipeline using an **ESP32 microcontroller**, an **MQ-2 gas sensor**, and a managed **EMQX Cloud MQTT broker**. The system continuously samples ambient gas concentrations, converts the analog data into meaningful Parts Per Million (PPM) values for LPG and Smoke, and securely streams the telemetry data over an encrypted TLS connection to a cloud platform for database ingestion and browser-based visualization.

---

## ⚙️ System Architecture & Data Flow
1. **Data Acquisition:** The MQ-2 sensor measures gas concentrations via internal chemical resistance variations. The data is passed to the ESP32 via a safe ADC1 channel (GPIO 32).
2. **Edge Processing:** The ESP32 processes the raw 12-bit ADC values, computes the algorithmic logarithmic equations derived from the sensor datasheet, and compiles a structured JSON data packet.
3. **Network Resilience:** A robust, non-blocking Wi-Fi State Machine runs concurrently to manage link dropouts and handle seamless automatic reconnection routines without pausing data loops.
4. **Cloud Transmission:** The payload is transmitted securely via **MQTTS (Port 8883 with TLS/SSL)** to an EMQX Cloud Broker instance located in the `asia-southeast1` region.
5. **Consumption & Storage:** The data stream can be visualized live using secure WebSockets (`wss://` port 8084) via MQTTX Web and routed smoothly into the assigned system database.

---

## 📂 Repository Directory Structure
```text
├── hardware/
│   └── database-assignment.ino   # Fully integrated ESP32 source code
├── database/
│   └── schema.sql               # Database schemas and structural scripts
└── README.md                    # Project documentation

## ⚙️ System Architecture & Data Flow

```mermaid
graph TD
    %% Define Styles
    classDef env fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef HW fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef cloud fill:#efebe9,stroke:#5d4037,stroke-width:2px;
    classDef view fill:#e8f5e9,stroke:#388e3c,stroke-width:2px;

    %% Workflow Nodes
    Env[Physical Environment] -->|Gas Particles React| MQ2[MQ-2 Gas Sensor Module]
    MQ2 -->|Analog Voltage: 0V - 3.3V| ESP32[ESP32 Microcontroller Core]
    
    subgraph Edge Processing [ESP32 Main Loop]
        ESP32 --> ADC[Read 12-bit ADC Pin 32]
        ADC --> Math[Calculate Voltage & PPM Log Curves]
        Math --> JSON[Wrap Data into JSON Payload]
        WM[Wi-Fi State Machine] -->|Monitor & Reconnect| ESP32
    end
    
    JSON -->|Secure MQTTS Port 8883| EMQX[EMQX Cloud Broker Instance]
    
    subgraph Data Consumption & Ingestion
        EMQX -->|Secure WebSockets Port 8084| MQTTX[MQTTX Web Browser Interface]
        EMQX -->|Data Integration / Rule Engine| DB[(Target Assignment Database)]
    end

    %% Apply Styles
    class Env env;
    class MQ2,ESP32,ADC,Math,JSON,WM HW;
    class EMQX cloud;
    class MQTTX,DB view;
