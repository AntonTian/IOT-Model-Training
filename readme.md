# 🗑️ SmartBin IoT Time-Series Predictor

A robust Internet of Things (IoT) system designed to predict the time remaining before a smart trash bin reaches full capacity. This project integrates ESP32 hardware, a LightGBM regression model, and Cloud infrastructure to provide real-time visibility and historical analytics.

## 🏗️ System Architecture

The project implements a polling-based microservices architecture:

1. **Hardware Layer (ESP32 + Ultrasonic Sensor):** Reads the physical garbage fill level and transmits the raw telemetry to the Blynk IoT Cloud.
2. **Real-Time Dashboard (Blynk):** Acts as the primary UI to monitor current capacity, pickup event status, and display the predicted time-to-full.
3. **Prediction Backend (Python on Hugging Face):** A continuous background worker that polls data via Blynk's REST API, executes real-time feature engineering (Lag & Cyclical Time features), runs AI inference, and pushes predictions back to the dashboard.
4. **Time-Series Database (InfluxDB Cloud):** Archives raw telemetry and prediction logs for historical trend analysis.

## 📊 Model Performance

The core prediction engine utilizes a **LightGBM Regressor** optimized via Bayesian Optimization using **Optuna**. 

To strictly prevent data leakage, the model was evaluated using a **Temporal Split** strategy (trained on past data, tested on future data) combined with `GroupKFold` to ensure robustness across different bin locations.

**Evaluation Metrics (16-Day Dataset):**
* **Best Cross-Validation MAE:** 1.6717%
* **Overall Test MAE:** 1.5928%
* **High-Fill Test MAE (Capacity $\ge 75\%$):** 1.6012%
* **Overall RMSE:** 2.0123%

> *Note: An error rate of ~1.5% is equivalent to a physical deviation of 1.5 cm in a 100 cm bin, which effectively reaches the physical accuracy limits of the HC-SR04/JSN-SR04T ultrasonic sensors.*

## ⚙️ Getting Started

It is recommended to use a virtual environment (e.g., `venv` or `conda`) running Python 3.9+.

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/smartbin-iot-predictor.git
cd smartbin-iot-predictor

# 2. Create and activate a Virtual Environment
python -m venv venv

# On Windows:
.\venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt
```

## 🚀 Usage

### 1. Running the Backend Server
The `app.py` script runs a Flask Web Server (serving as a keep-alive mechanism for Hugging Face Spaces) alongside an IoT polling Background Thread.

Ensure your API credentials (`BLYNK_AUTH_TOKEN`, `INFLUX_URL`, `INFLUX_TOKEN`, `INFLUX_ORG`, `INFLUX_BUCKET`) are configured inside the script or as environment variables before running.

```bash
python app.py
```

### 2. Viewing the Optuna Dashboard
We use `optuna-dashboard` to visualize the hyperparameter tuning history and parameter importances. The tuning history is persisted in a local SQLite database.

```bash
optuna-dashboard sqlite:///optuna_smartbin_study.sqlite3
```
Open your browser and navigate to `http://127.0.0.1:8080/`. 

*Tip: Enable the **Log y scale** toggle in the History chart to properly observe the convergence of the Tree-structured Parzen Estimator (TPE) algorithm.*