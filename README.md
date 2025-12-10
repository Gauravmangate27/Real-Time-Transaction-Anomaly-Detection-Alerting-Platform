# Real-Time Transaction Anomaly Detection & Alerting Platform

A production-style fintech analytics platform that ingests transaction streams via Kafka, detects anomalous / potentially fraudulent transactions using machine learning, and exposes real-time alerts via REST APIs and dashboards.

---

## 🚀 Key Features

- Real-time ingestion of transaction events using **Apache Kafka** and **Spark Streaming**. 
- Feature engineering and context-based behavioral profiling per user/card.  
- ML-powered anomaly detection (unsupervised / supervised models).
- Risk classification and rule-based post-processing for business-friendly alerts.
- REST API + dashboard for viewing and investigating suspicious transactions in real time.  

---

## 🧱 Tech Stack

- **Streaming & Ingestion:** Apache Kafka, Apache Spark Streaming. 
- **ML & Analytics:** Python (scikit-learn ).  
- **Storage:**  MongoDB (alerts & history), object storage for models.  
- **Backend API:** Flask.  
- **Monitoring (optional):** Prometheus, Grafana.

---

## 🔁 End-to-End Workflow

The platform is organized into two main pipelines:

1. **Streaming Ingestion & Feature Engineering Pipeline**  
2. **Real-Time Anomaly Detection & Alerting Pipeline**

---

### 1️⃣ Streaming Ingestion & Feature Engineering Pipeline

1. **Event capture (clients → Kafka)**  
   - Upstream payment systems or mock scripts send transaction events (user_id, card_id, amount, merchant, location, timestamp, channel, etc.) to a Kafka topic such as `transactions_raw`.  
   - Events are encoded as JSON for easy downstream consumption.  

2. **Real-time preprocessing (Kafka → Spark Streaming)**  
   - Spark Streaming / Structured Streaming consumes messages from `transactions_raw` in micro-batches.  
   - Cleans records (type casting, missing values, currency normalization) and enriches them with derived attributes:  
     - Time features: hour of day, day of week, weekend flag.  
     - Location/device features: country, distance from last known location, channel type.   

3. **Behavioral feature aggregation**  
   - Maintain stateful aggregates per user/card:  
     - Rolling average transaction amount.  
     - Count and total amount over last X minutes/hours/days.  
     - Category-wise spend distributions (e.g., food, travel, shopping).  
   - Build a feature vector for each transaction combining raw fields + aggregates.  

4. **Feature vector emission**  
   - Emit enriched feature vectors to a downstream Kafka topic such as `transactions_features` for scoring.
   - Optionally persist a copy to storage (e.g., data lake) for offline training and analysis.

5. **Offline model training (batch)**  
   - Use historical feature data to train anomaly detection models:  
     - Unsupervised: Isolation Forest, One-Class SVM, autoencoders when labels are limited. 
     - Supervised: Gradient boosting / Random Forest if labelled fraud data is available. 
   - Save the trained model and configuration into a model registry or object store for online serving.]

---

### 2️⃣ Real-Time Anomaly Detection & Alerting Pipeline

1. **Online scoring service**  
   - A Spark Streaming job or a Python microservice subscribes to `transactions_features`.
   - Loads the latest model from the registry into memory.  
   - For each incoming feature vector, computes:  
     - An **anomaly score** (continuous measure).  
     - An **anomaly label** (normal / anomalous) using a configurable threshold.

2. **Risk classification & business rules**  
   - Map raw anomaly scores to risk levels such as `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`.  
   - Apply business rules:  
     - Flag very high-value anomalies as **CRITICAL**.  
     - Auto-approve low-value, low-score transactions.  
     - Route selected anomalies for manual review.

3. **Alert routing & persistence**  
   - Publish flagged anomalies to a dedicated Kafka topic (e.g., `transactions_anomalies`) to decouple consumers. 
   - Persist anomalies (and relevant features) to a database (PostgreSQL / MongoDB) for long-term storage, dashboards, and audits.

4. **REST API & dashboard**  
   - A FastAPI/Flask backend exposes endpoints to:  
     - Fetch recent anomalies by user, risk level, or time window.  
     - Inspect transaction details, feature values, and model outputs.  
   - A simple dashboard visualizes:  
     - Real-time stream of anomalies.  
     - Risk distribution by geography, merchant, and time-of-day.  
     - Historical trends in anomaly rates and false positives.  

5. **Explainability & feedback loop**  
   - Use model-specific techniques (e.g., feature importances, SHAP) to show which features contributed most to the anomaly score.  
   - Allow reviewers to label each alert as **true fraud** or **false positive**, and store this feedback.
   - Regularly retrain models and adjust thresholds using this labelled data.  

6. **Monitoring & reliability**  
   - Track key metrics:  
     - End-to-end latency from transaction ingestion to alert.  
     - Throughput (transactions/sec), Kafka lag, Spark batch duration.
     - Model performance metrics where labels exist (precision, recall, false positive rate). 
   - Surface metrics in Prometheus/Grafana or similar tools for production-style observability.  


