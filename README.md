# TECHIN515 Lab 5 - Edge-Cloud Offloading

This project implements an edge-cloud offloading strategy for gesture recognition using an ESP32-based magic wand. The system performs inference locally on the ESP32 and offloads to Microsoft Azure cloud when confidence is low.

## Project Structure
```
.
├── ESP32_to_cloud/             # ESP32 Arduino code
│   └── ESP32_to_cloud.ino      # Main ESP32 sketch
├── trainer_scripts             # Scripts
    ├── train.ipynb                 # Model training script
    ├── model_register.ipynb        # Model register script
├── app/                        # Web app for model deployment
    ├── wand_model.h5               # trained model
    ├── app.py                      # Script of web app
    ├── requirements.txt            # Dependencies required by web app
└── data/                       # Training data directory
    ├── O/                           # O-shape gesture samples
    ├── V/                           # V-shape gesture samples
    ├── Z/                           # Z-shape gesture samples
    └── some_class/                  # Some other gesture samples
```

## Hardware Requirements
- Magic wand from Lab 4
- ESP32 board

## Software Requirements
- Arduino IDE with ESP32 board support
- Required libraries:
  - Adafruit MPU6050
  - Adafruit Sensor
  - Wire (built-in)
- Microsoft Azure account
- Python 3.8 or newer
- Required Python packages (see requirements.txt)

## Setup Instructions

1. **Microsoft Azure Setup**
   - Create a Resource Group
   - Create a Machine Learning Workspace
   - Create a Compute Instance
   - Host training data in Azure Blob
   - Train and register the model

2. **Web App Deployment**
   - Navigate to app directory
   - Install dependencies: `pip install -r requirements.txt`
   - Run the server: `python app.py`

3. **ESP32 Configuration**
   - Update WiFi credentials in ESP32_to_cloud.ino
   - Set server URL to your web app URL
   - Configure confidence threshold for offloading

## Discussion Questions

### 1. Server vs. Local Confidence
**Question:** Is server's confidence always higher than wand's confidence from your observations? What is your hypothetical reason for the observation?

**Answer:** The server's confidence is not always higher than the wand's confidence. This can be attributed to several factors:
1. The server model might be trained on a more diverse dataset, potentially including merged data from multiple students
2. The server has more computational resources, allowing for more complex model architectures
3. However, the wand's local model might perform better for specific gestures it was trained on, especially if the training data was collected in similar conditions to the testing environment
4. Network latency and data transmission quality can affect the confidence scores when offloading to the server

### 2. Data Flow
**Question:** Sketch the data flow of this lab.

**Answer:** The data flow in this lab follows these steps:
1. **Data Collection and Training**
   - Collect gesture data using the ESP32 magic wand
   - Upload data to Azure Blob storage
   - Train model in Azure ML workspace
   - Register model in Azure ML

2. **Inference Flow**
   - ESP32 captures sensor data from MPU6050
   - Local inference is performed first
   - If confidence < threshold:
     - Raw sensor data is sent to Azure web app
     - Server performs inference
     - Results are sent back to ESP32
   - If confidence >= threshold:
     - Local inference result is used directly
   - LED feedback is provided based on the final result

### 3. Edge-First Strategy Analysis
**Question:** Our approach is edge-first, fallback-to-server when uncertain. Analyze pros and cons of this approach from the following aspects:
- Reliance on connectivity
- Latency
- Prediction consistency
- Data privacy

**Answer:**
**Pros:**
- **Connectivity:** Less dependent on constant internet connection since most inferences are done locally
- **Latency:** Lower latency for high-confidence predictions as they don't require network communication
- **Prediction Consistency:** More consistent performance for well-known gestures
- **Data Privacy:** Most sensitive data (sensor readings) stays on device unless offloaded

**Cons:**
- **Connectivity:** Still requires internet for uncertain cases, making it unreliable in poor network conditions
- **Latency:** Higher latency for uncertain cases due to network round-trip time
- **Prediction Consistency:** Potential inconsistency between local and cloud predictions
- **Data Privacy:** Some data is still sent to the cloud, raising privacy concerns

### 4. Mitigation Strategy
**Question:** Name a strategy to mitigate at least one limitation named in question 3.

**Answer:** To mitigate the connectivity and latency issues, we can implement a hybrid caching strategy:
1. When the device is online, maintain a local cache of recent cloud predictions
2. For uncertain cases when offline, use the cached predictions if available
3. Implement a confidence threshold for cache usage (e.g., only use cached predictions if they were above 90% confidence)
4. Periodically update the cache when online to ensure predictions stay current
5. This approach reduces dependency on constant connectivity while maintaining reasonable prediction quality

## Clean Up
Remember to clean up Azure resources when you're done to avoid unnecessary charges:
1. Go to resource groups from portal menu
2. Delete the resource group
3. Wait for the operation to complete

## Deliverables
- GitHub repository with complete project
- Pictures of serial monitor showing:
  - Local inference with high confidence
  - Cloud inference when confidence is low
- Report of all discussion questions (included in this README)