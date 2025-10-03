# System Architecture

## Overview

This document describes the hybrid, multi-tiered architecture for a distributed robotics control system. The architecture is designed to leverage the strengths of different computing platforms at various layers, from edge devices for real-time control to cloud services for advanced AI processing.

## Architecture Layers

### 1. Edge Layer (Pico W + Arduino)

The Edge Layer consists of embedded hardware responsible for low-latency, real-time operations:

#### Raspberry Pi Pico W
- **Primary Role**: Real-time control and network communication
- **Responsibilities**:
  - Motor control with minimal latency
  - Sensor data acquisition and processing
  - Wi-Fi connectivity for communication with Control Layer
  - Real-time decision making for safety-critical operations
  - Direct hardware interfacing via GPIO pins
- **Communication**: Wi-Fi connection to Desktop PC (Control Layer)

#### Arduino
- **Primary Role**: Secondary mechanical and sensor management
- **Responsibilities**:
  - Additional sensor interfacing
  - Backup control systems
  - Specialized hardware protocols (I2C, SPI, etc.)
  - Offloading specific real-time tasks from Pico W
- **Communication**: Serial or I2C connection to Pico W

**Key Characteristics**:
- Ultra-low latency (microsecond to millisecond response times)
- Direct hardware access
- Minimal processing overhead
- Reliable real-time operation

---

### 2. Control Layer (Desktop PC)

The Control Layer serves as the computational brain of the system:

#### Desktop PC
- **Primary Role**: High-level logic, AI processing, and coordination
- **Responsibilities**:
  - Complex decision-making algorithms
  - AI model inference (local models)
  - Sensor data aggregation and analysis
  - Motion planning and path optimization
  - User interaction handling
  - System state management
  - Communication orchestration between all layers
  - Local AI processing (computer vision, ML inference)
- **Communication**: 
  - Wi-Fi to Edge Layer (Pico W)
  - HTTP/WebSocket APIs to Cloud Layer (GCP)
  - Display output to UI Layer via spacedesk

**Key Characteristics**:
- High computational power for AI/ML workloads
- Moderate latency (milliseconds to seconds)
- Flexible software environment
- Central coordination point

---

### 3. UI Layer (Lenovo Tab M11)

The UI Layer provides user visualization and interaction:

#### Lenovo Tab M11 Tablet
- **Primary Role**: Wireless monitoring and display
- **Responsibilities**:
  - Custom dashboard visualization
  - Real-time system status display
  - AI avatar rendering and interaction
  - User input collection (touch interface)
  - Alerts and notifications display
  - System diagnostics visualization
- **Communication**: Receives display via spacedesk from Desktop PC

**Key Characteristics**:
- Mobile and flexible positioning
- Touch-based interaction
- Real-time visualization
- Low power consumption
- Wireless operation

---

### 4. Cloud Layer (Google Cloud Platform)

The Cloud Layer provides advanced, compute-intensive AI capabilities:

#### GCP Services
- **Primary Role**: Advanced AI processing and data storage
- **Responsibilities**:
  - Object recognition and classification
  - Natural language processing
  - Advanced computer vision tasks
  - Machine learning model training
  - Historical data storage and analytics
  - Remote monitoring and diagnostics
- **Communication**: API calls from Desktop PC (Control Layer)

**Potential GCP Services Used**:
- Cloud Vision API for object recognition
- Cloud AI Platform for ML model deployment
- Cloud Storage for data persistence
- Cloud Pub/Sub for event-driven architecture
- Cloud Functions for serverless processing

**Key Characteristics**:
- Unlimited computational resources
- State-of-the-art AI models
- Higher latency (hundreds of milliseconds to seconds)
- Internet connectivity required
- Pay-per-use model

---

## System Communication Flow

```
┌─────────────────────────────────────────────────────────────┐
│                        Cloud Layer                          │
│                  (Google Cloud Platform)                    │
│  - Object Recognition  - ML Training  - Data Storage        │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS/REST APIs
                         │
┌────────────────────────▼────────────────────────────────────┐
│                      Control Layer                          │
│                     (Desktop PC)                            │
│  - High-level Logic  - AI Processing  - Coordination        │
└─────┬───────────────────────────────────────────────┬───────┘
      │ Wi-Fi                                         │ spacedesk
      │                                               │
┌─────▼────────────────┐                    ┌────────▼─────────┐
│    Edge Layer        │                    │    UI Layer      │
│  (Pico W + Arduino)  │                    │  (Lenovo Tab M11)│
│  - Motor Control     │                    │  - Dashboard     │
│  - Sensors           │                    │  - AI Avatar     │
│  - Real-time Tasks   │                    │  - Monitoring    │
└──────────────────────┘                    └──────────────────┘
```

---

## Data Flow Patterns

### 1. Sensor to Decision Flow
1. **Edge Layer**: Sensors capture raw data (cameras, IMU, etc.)
2. **Edge Layer**: Basic filtering and preprocessing
3. **Control Layer**: Data aggregation and analysis
4. **Control Layer**: Decision-making logic
5. **Control Layer**: (Optional) Query Cloud Layer for advanced AI
6. **Control Layer**: Send commands back to Edge Layer
7. **Edge Layer**: Execute motor/actuator commands

### 2. Visualization Flow
1. **Edge Layer**: Collect real-time sensor data
2. **Control Layer**: Aggregate and format data
3. **Control Layer**: Render dashboard and AI avatar
4. **UI Layer**: Display via spacedesk wireless protocol

### 3. AI Processing Flow
1. **Edge Layer**: Capture image/sensor data
2. **Control Layer**: Initial processing and filtering
3. **Control Layer**: Send to Cloud Layer if advanced AI needed
4. **Cloud Layer**: Perform object recognition/ML inference
5. **Cloud Layer**: Return results to Control Layer
6. **Control Layer**: Integrate results into decision-making
7. **Control Layer**: Send commands to Edge Layer

---

## Latency Characteristics

| Operation Type | Layer | Expected Latency |
|---------------|-------|------------------|
| Motor Control | Edge | < 1 ms |
| Sensor Reading | Edge | < 10 ms |
| Local AI Inference | Control | 10-100 ms |
| Display Update | UI | 16-33 ms (30-60 FPS) |
| Cloud AI Query | Cloud | 200-2000 ms |

---

## Failure Modes and Resilience

### Edge Layer Failures
- **Arduino Failure**: Pico W takes over secondary functions
- **Wi-Fi Loss**: Pico W operates in autonomous safe mode
- **Power Loss**: Emergency stop procedures activated

### Control Layer Failures
- **PC Crash**: Edge Layer enters safe mode with basic autonomous operation
- **Network Loss to Cloud**: Fall back to local AI models only

### UI Layer Failures
- **Tablet Disconnection**: System continues operating without visualization
- **spacedesk Failure**: Alternative monitoring via Control Layer directly

### Cloud Layer Failures
- **API Unavailable**: Use cached results or fallback to local AI models
- **Rate Limiting**: Queue requests and prioritize critical operations

---

## Security Considerations

1. **Edge Layer**: 
   - Secure Wi-Fi with WPA3
   - Firmware signature verification
   - Physical security of hardware

2. **Control Layer**:
   - Local firewall configuration
   - API key management
   - Encrypted communication channels

3. **Cloud Layer**:
   - OAuth 2.0 authentication
   - API key rotation
   - Encrypted data transmission (TLS)
   - Data privacy compliance

---

## Scalability

The architecture is designed to scale in multiple dimensions:

- **Horizontal Scaling**: Add more Edge devices (Pico W units) for complex robots
- **Vertical Scaling**: Upgrade Desktop PC for more powerful AI processing
- **Cloud Scaling**: Leverage GCP auto-scaling for variable workloads
- **UI Scaling**: Support multiple tablet displays for different views

---

## Development and Deployment

### Edge Layer Development
- Language: MicroPython or C/C++ (Arduino)
- IDE: Thonny, Arduino IDE, or VS Code with extensions
- Testing: Unit tests on PC, integration tests on hardware

### Control Layer Development
- Language: Python, Node.js, or C++
- Frameworks: TensorFlow, PyTorch, OpenCV
- Testing: Unit tests, integration tests, simulation environments

### UI Layer Development
- Technology: Web-based (HTML/CSS/JS) or native mobile frameworks
- Display: spacedesk client on Android
- Testing: Browser testing, responsive design validation

### Cloud Layer Development
- Platform: GCP Console, gcloud CLI
- APIs: REST/gRPC interfaces
- Testing: API testing, load testing, cost optimization

---

## Future Enhancements

1. **Edge Intelligence**: Deploy lightweight AI models directly on Pico W
2. **Mesh Networking**: Enable direct communication between multiple Edge devices
3. **Predictive Maintenance**: Use Cloud Layer ML to predict hardware failures
4. **Voice Control**: Integrate voice commands via Cloud Speech-to-Text
5. **Remote Operation**: Enable secure remote control over the internet
6. **Telemetry**: Comprehensive logging and analytics pipeline

---

## Conclusion

This hybrid, multi-tiered architecture provides a robust, scalable, and flexible foundation for a distributed robotics control system. By carefully allocating responsibilities across different computing layers, the system achieves optimal performance, from real-time control at the edge to advanced AI processing in the cloud, while maintaining a user-friendly interface for monitoring and interaction.
