# SALUS.MATERNA — AI & Solar-Powered Maternal Health Monitoring System

![Status](https://img.shields.io/badge/Status-Prototype%20Complete-success)
![Platform](https://img.shields.io/badge/Platform-ESP32%20%2B%20Azure-blue)
![Power](https://img.shields.io/badge/Power-Solar%20Powered-yellow)

---

## 🏥 Overview

SALUS.MATERNA is a solar-powered, IoT-enabled maternal health monitoring device designed for rural Kenyan healthcare facilities. It was developed to address the critical infrastructure gaps that contribute to Kenya's maternal mortality rate of **355 deaths per 100,000 live births** — equivalent to roughly 16 women every single day.

Rural facilities face three core, interconnected problems this system targets:
1. **No continuous monitoring** — intermittent manual checks every 4–6 hours miss early deterioration
2. **Frequent power outages** — blackouts render conventional electronic equipment non-functional
3. **No remote alerting** — understaffed clinics have no way to notify off-site clinicians of deteriorating patients

SALUS.MATERNA addresses all three simultaneously through continuous vital signs monitoring, solar-powered off-grid operation, real-time cloud telemetry, and remote SMS/voice call alerts — combined with an ensemble machine learning model for early detection of maternal complications including PPH, sepsis, hypertensive disorders, and pulmonary embolism.

---

## ✅ Key Features

- **Continuous vital signs monitoring** — Heart Rate, SpO₂, Temperature, and cuffless Blood Pressure
- **Novel cuffless BP measurement** — Piezoelectric + PPG sensors along the radial artery, using Pulse Wave Velocity derived from the Moens-Korteweg equation and Beer-Lambert PIR model
- **Solar-powered operation** — Full off-grid functionality via 6V PV panel and Li-ion battery
- **Cloud telemetry** — Real-time data upload to Microsoft Azure at 60-second intervals
- **Remote alerting** — Twilio SMS and voice calls delivered within 60 seconds of a detected risk
- **On-device alerts** — OLED display, LED indicators, and buzzer for local notification
- **Ensemble ML model** — Early detection of PPH, sepsis, hypertensive disorders, and pulmonary embolism
- **Web dashboard** — Real-time monitoring interface accessible from any device
- **Cost-effective** — Total hardware cost of approximately KES 6,170 (~USD 48)

---

## 📁 Project Structure

```
salus-materna/
│
├── 📄 README.md                         ← This file
│
├── 📘 Documentation/
│   ├── QUICK_START.md                   ← 30-minute setup guide
│   ├── SETUP_GUIDE.md                   ← Complete step-by-step guide
│   └── DEPLOYMENT_CHECKLIST.md         ← Verification checklist
│
├── 🐍 Python Scripts/
│   ├── azure_function_app.py            ← Azure Functions for ML inference & alerts
│   ├── 03_convert_to_append_blob.py     ← Dataset conversion (run before deploying)
│   └── requirements.txt                 ← Python dependencies
│
├── 🔧 ESP32 Code/
│   └── ESP32_Cloud_Integration.ino      ← Main Arduino firmware for ESP32
│
├── 🌐 Web Interface/
│   └── web_interface.html               ← Real-time monitoring dashboard
│
└── ⚙️ Configuration/
    └── host.json                         ← Azure Functions configuration
```

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────┐
│                    PATIENT BEDSIDE                   │
│                                                      │
│  ESP32 Microcontroller                               │
│  ├── MAX30102 (Heart Rate + SpO₂ via PPG)            │
│  ├── MLX90614ESF (Infrared Temperature)              │
│  ├── Piezoelectric Ceramic Sensor (cuffless BP)      │
│  ├── OLED Display (real-time readings)               │
│  ├── RGB LEDs (risk level indicator)                 │
│  └── Buzzer (local alarm)                            │
│                                                      │
│  Solar Power Subsystem                               │
│  ├── 6V Photovoltaic Panel                           │
│  └── 3.7V Li-ion Battery + Charging Circuit         │
└──────────────────────────┬───────────────────────────┘
                           │ Wi-Fi (HTTPS)
                           ▼
        ┌──────────────────────────────────────┐
        │         MICROSOFT AZURE              │
        │                                      │
        │  Azure Functions (Serverless)        │
        │  ├── ML inference (risk prediction)  │
        │  ├── Alert trigger logic             │
        │  └── Patient data management         │
        │                                      │
        │  Azure Blob Storage                  │
        │  ├── Time-series vitals dataset      │
        │  └── Trained ML ensemble model (.pkl)│
        │                                      │
        │  Azure Static Web Apps               │
        │  └── Real-time monitoring dashboard  │
        └────────────┬─────────────────────────┘
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   ┌─────────────┐      ┌──────────────┐
   │   Twilio    │      │  Web Browser │
   │ SMS + Voice │      │  Dashboard   │
   │   Alerts    │      │  (any device)│
   └──────┬──────┘      └──────────────┘
          ▼
   📱 Healthcare Provider
```

---

## 🔬 How It Works

### 1. Continuous Sensing (ESP32)
Every second, the ESP32 reads all vital signs from its sensor array. Readings are averaged, displayed on the OLED screen, and buffered locally.

### 2. Cuffless Blood Pressure Measurement
Blood pressure is estimated by combining a piezoelectric ceramic sensor and the MAX30102 PPG sensor, both placed along the radial artery. The system calculates **Pulse Transit Time (PTT)** and **Pulse Wave Velocity (PWV)** between the two sensors, then derives SBP and DBP using the **Moens-Korteweg equation** and the **Beer-Lambert Peak Intensity Ratio (PIR)** model. A one-time personal calibration is required.

### 3. Cloud Upload & ML Inference (Every 60 seconds)
The ESP32 sends a JSON payload to Azure Functions containing the patient's averaged vitals. The Azure Function loads the trained ensemble ML model, preprocesses the data, runs inference, and returns a risk level:

| Risk Level | Threshold | Response |
|---|---|---|
| LOW | < 0.3 | Blue LED |
| MODERATE | 0.3 – 0.6 | Yellow LED |
| HIGH | > 0.6 | Red LED + Buzzer + SMS/Call |

Additional alerts are triggered for finger-off detection (patient contact lost) and for individual vital signs crossing clinical thresholds (e.g., SBP ≥ 140 mmHg).

### 4. Remote Alerts (Twilio)
On a HIGH risk flag or abnormal reading, Twilio simultaneously fires an SMS and a voice call to the registered healthcare provider, typically within 60 seconds of detection.

### 5. Web Dashboard
The Azure Static Web Apps dashboard displays real-time vitals, the current risk assessment, and a full alert history log. It updates automatically and is accessible from any device.

---

## 📊 Sensor Validation Results

All sensors were validated against clinical-grade reference equipment.

### SpO₂ (MAX30102 vs. Contec CMS50D+)
- **Mean Squared Error: 1.7**
- Most readings matched the reference within ±1%
- Maximum deviation: −3%

### Temperature (MLX90614ESF vs. Braun Clinical Thermometer)
- **Mean Squared Error: 0.141°C**
- Majority of readings within ±0.3°C of reference
- Maximum deviation: 0.7°C

### Cuffless Blood Pressure (Piezo + MAX30102 vs. Cuff-Based Monitor)
| Measurement | SBP % Error | DBP % Error |
|---|---|---|
| 1 | 3.28% | 5.00% |
| 2 | 3.62% | 4.55% |
| 3 | 3.48% | 4.17% |
| 4 | 3.13% | 4.76% |
| 5 | 3.52% | 4.35% |
| 6 | 3.39% | 3.95% |
| 7 | 3.79% | 5.81% |
| 8 | 4.05% | 5.21% |
| 9 | 3.23% | 4.88% |
| 10 | 3.68% | 4.44% |
| **Mean** | **3.52%** | **4.71%** |

### Cloud & Alert Performance
- **Data transmission success rate**: 98.7% (60-second intervals)
- **Dashboard update latency**: < 3 seconds
- **Alert delivery time**: Within 60 seconds
- **Solar subsystem**: Full off-grid operation confirmed

---

## 🎯 Complication Detection Logic

The ML ensemble model monitors for four primary postpartum complications, each associated with a unique physiological signature in the continuous vital signs data:

| Complication | Key Vital Sign Patterns |
|---|---|
| **Postpartum Hemorrhage (PPH)** | Progressive tachycardia + hypotension |
| **Sepsis** | Tachycardia + fever + SpO₂ decline |
| **Hypertensive Disorders** | Rising SBP/DBP trend |
| **Pulmonary Embolism** | SpO₂ desaturation + tachycardia |

---

## 🛠️ Technical Specifications

### Hardware
| Component | Part | Purpose |
|---|---|---|
| Microcontroller | ESP32 | Processing, Wi-Fi, firmware |
| Pulse Oximetry + PPG | MAX30102 | HR, SpO₂, PPG waveform for BP |
| Infrared Thermometer | MLX90614ESF | Non-contact temperature |
| Pressure Sensor | Piezoelectric Ceramic | Pulse wave detection for BP |
| Display | SH1106 OLED (128×64) | Local readings display |
| Indicators | RGB LEDs + Buzzer | Local alerts |
| Solar Panel | 6V PV Panel | Primary power source |
| Battery | 3.7V Li-ion | Energy storage for off-grid use |

### Cloud Infrastructure
| Service | Purpose |
|---|---|
| Azure Functions (Python 3.9) | ML inference + alert logic |
| Azure Blob Storage | Time-series dataset + model storage |
| Azure Static Web Apps | Real-time monitoring dashboard |
| Twilio API | SMS + voice call alerts |

### ML Model
- **Type**: Scikit-learn ensemble (XGBoost + Random Forest + additional classifiers)
- **Training performance**: 85–92% accuracy
- **Sensitivity (Recall)**: ~88%
- **Specificity**: ~84%
- **Continuous learning**: Each uploaded reading appended to training dataset

---

## 💰 Cost

### Hardware (One-Time)
| Item | Approx. Cost (KES) |
|---|---|
| ESP32 | 650 |
| MAX30102 | 400 |
| MLX90614ESF | 800 |
| Piezoelectric Sensor | 200 |
| OLED Display | 500 |
| Solar Panel + Battery + Charging Circuit | 2,500 |
| LEDs, Buzzer, Wiring, Enclosure | 300 |
| **Total** | **~KES 6,170 (~USD 48)** |

### Monthly Operational Costs (Azure + Twilio)
- Estimated at **< $5 USD/month** at single-device scale

---

## 🚀 Setup Guide

### Prerequisites
- [ ] Microsoft Azure account
- [ ] Twilio account (for SMS/voice alerts)
- [ ] Python 3.8+ installed
- [ ] Arduino IDE installed
- [ ] ESP32 hardware assembled with sensors

### Step 1: Convert Dataset
```bash
python 03_convert_to_append_blob.py
```
Wait for the confirmation message before proceeding.

### Step 2: Deploy Azure Function
1. Create a Function App in the Azure Portal
2. Add your Twilio credentials and Azure Storage connection string as **Application Settings** (never hardcode these)
3. Deploy `azure_function_app.py`
4. Copy the Function URL

### Step 3: Deploy Web Interface
1. Update `web_interface.html` with your Function URL
2. Deploy to Azure Static Web Apps
3. Note the generated web URL

### Step 4: Flash ESP32
1. Open `ESP32_Cloud_Integration.ino` in Arduino IDE
2. Update the Wi-Fi credentials and Azure Function URL in the configuration section
3. Upload to ESP32 at 115200 baud
4. Monitor Serial output for confirmation

### Step 5: Calibrate Blood Pressure
A one-time personal calibration against a reference cuff-based monitor is required before BP measurements are accurate for a given patient.

### Step 6: Test
1. Enter patient info via Serial: `new:PATIENT-ID,AGE`
2. Place finger on MAX30102 sensor
3. Allow 45 seconds for stabilization
4. Verify cloud communication in Serial Monitor
5. Confirm LED risk level indicator
6. Verify SMS/call alert on HIGH risk

📖 See `QUICK_START.md` for full details and `SETUP_GUIDE.md` for troubleshooting.

---

## 🔒 Security & Privacy

- All data transmitted over **HTTPS/TLS 1.2+**
- Azure Blob Storage **encrypted at rest**
- Credentials (Twilio, Azure Storage, Wi-Fi) should be stored as **Azure Application Settings** or in a `.env` file — **never committed to version control**
- Access controlled via **Azure RBAC**
- Architecture supports **HIPAA compliance** with appropriate configuration (BAA with Microsoft, audit logging, data retention policies)


---

## 🧰 Maintenance

| Frequency | Task |
|---|---|
| Daily | Check system uptime, review alert logs |
| Weekly | Verify data uploads, check storage usage |
| Monthly | Review ML model performance, check sensor accuracy |
| Quarterly | Retrain ML model with accumulated data |
| Hardware | Clean sensors weekly; inspect wiring monthly |

---

## 🗺️ Roadmap

### Version 1.0 (Current — Prototype)
✅ Continuous vital signs monitoring (HR, SpO₂, Temp, BP)  
✅ Solar-powered off-grid operation  
✅ Azure cloud telemetry  
✅ Twilio SMS + voice call alerts  
✅ Ensemble ML risk detection  
✅ Real-time web dashboard  

### Version 2.0 (Planned)
🔄 Multi-patient monitoring on a single dashboard  
🔄 Predictive alerts (hours-ahead forecasting)  
🔄 Miniaturization for wearability  
🔄 Expanded clinical validation study  

### Version 3.0 (Future)
🔮 Mobile app (iOS/Android)  
🔮 EHR system integration  
🔮 Federated learning across facilities  
🔮 Regulatory approval pathway (KEBS, FDA/CE)  

---

## 📚 References

1. World Health Organization. *Trends in Maternal Mortality 2000–2020*. WHO, 2023.
2. Kenya National Bureau of Statistics. *Kenya Demographic and Health Survey*, 2022.
3. United Nations. *Sustainable Development Goal 3.1 — Maternal Mortality*. UN, 2015.
4. Kenya Ministry of Health. *Maternal Mortality Audit Reports, County Hospitals 2020–2023*.
5. MAX30102 Datasheet — Maxim Integrated / Analog Devices.
6. MLX90614 Datasheet — Melexis.
7. Microsoft Azure Functions Documentation — Microsoft.
8. ESP32 Technical Reference Manual — Espressif Systems.
9. Moens, I.A. & Korteweg, D.J. — Pulse Wave Velocity derivation for arterial stiffness estimation.
10. Beer-Lambert Law — optical absorption model for PPG-based arterial diameter change estimation.
*Developer: Cedric Murage Mwangi | Reg No: J23/5508/2020*  
*Last Updated: February 2026 | Version: 1.0*
