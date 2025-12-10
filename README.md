# Real-Time Transaction Anomaly Detection & Alerting Platform

A production-style fintech analytics platform that ingests transaction streams via Kafka, detects anomalous / potentially fraudulent transactions using machine learning, and exposes real-time alerts via REST APIs and dashboards.[web:130][web:140]

---

## 🚀 Key Features

- Real-time ingestion of transaction events using **Apache Kafka** and **Spark Streaming**.[web:136][web:139]  
- Feature engineering and context-based behavioral profiling per user/card.  
- ML-powered anomaly detection (unsupervised / supervised models). [web:130][web:149]  
- Risk classification and rule-based post-processing for business-friendly alerts.[web:133][web:146]  
- REST API + dashboard for viewing and investigating suspicious transactions in real time.  

---

## 🧱 Tech Stack

- **Streaming & Ingestion:** Apache Kafka, Apache Spark Streaming.[web:136]  
- **ML & Analytics:** Python (scikit-learn / PySpark ML, optionally XGBoost). [web:130]  
- **Storage:** PostgreSQL / MongoDB (alerts & history), object storage for models.  
- **Backend API:** FastAPI / Flask.  
- **Monitoring (optional):** Prometheus, Grafana / Kibana.[web:136][web:139]  

---

## 🔁 End-to-End Workflow

The platform is organized into two main pipelines:

1. **Streaming Ingestion & Feature Engineering Pipeline**  
2. **Real-Time Anomaly Detection & Alerting Pipeline**

---

### 1️⃣ Streaming Ingestion & Feature Engineering Pipeline

1. **Event capture (clients → Kafka)**  
   - Upstream payment systems or mock scripts send transaction events (user_id, card_id, amount, merchant, location, timestamp, channel, etc.) to a Kafka topic such as `transactions_raw`.[web:133][web:146]  
   - Events are encoded as JSON for easy downstream consumption.[web:140]  

2. **Real-time preprocessing (Kafka → Spark Streaming)**  
   - Spark Streaming / Structured Streaming consumes messages from `transactions_raw` in micro-batches.[web:136][web:139]  
   - Cleans records (type casting, missing values, currency normalization) and enriches them with derived attributes:  
     - Time features: hour of day, day of week, weekend flag.  
     - Location/device features: country, distance from last known location, channel type. [web:130][web:149]  

3. **Behavioral feature aggregation**  
   - Maintain stateful aggregates per user/card:  
     - Rolling average transaction amount.  
     - Count and total amount over last X minutes/hours/days.  
     - Category-wise spend distributions (e.g., food, travel, shopping). [web:140]  
   - Build a feature vector for each transaction combining raw fields + aggregates.  

4. **Feature vector emission**  
   - Emit enriched feature vectors to a downstream Kafka topic such as `transactions_features` for scoring.[web:136]  
   - Optionally persist a copy to storage (e.g., data lake) for offline training and analysis.[web:130]  

5. **Offline model training (batch)**  
   - Use historical feature data to train anomaly detection models:  
     - Unsupervised: Isolation Forest, One-Class SVM, autoencoders when labels are limited.[web:130][web:149]  
     - Supervised: Gradient boosting / Random Forest if labelled fraud data is available.[web:137]  
   - Save the trained model and configuration into a model registry or object store for online serving.[web:140]  

---

### 2️⃣ Real-Time Anomaly Detection & Alerting Pipeline

1. **Online scoring service**  
   - A Spark Streaming job or a Python microservice subscribes to `transactions_features`.[web:136][web:142]  
   - Loads the latest model from the registry into memory.  
   - For each incoming feature vector, computes:  
     - An **anomaly score** (continuous measure).  
     - An **anomaly label** (normal / anomalous) using a configurable threshold.[web:140][web:149]  

2. **Risk classification & business rules**  
   - Map raw anomaly scores to risk levels such as `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`.  
   - Apply business rules:  
     - Flag very high-value anomalies as **CRITICAL**.  
     - Auto-approve low-value, low-score transactions.  
     - Route selected anomalies for manual review.[web:133][web:146]  

3. **Alert routing & persistence**  
   - Publish flagged anomalies to a dedicated Kafka topic (e.g., `transactions_anomalies`) to decouple consumers.[web:140]  
   - Persist anomalies (and relevant features) to a database (PostgreSQL / MongoDB) for long-term storage, dashboards, and audits.[web:130][web:136]  

4. **REST API & dashboard**  
   - A FastAPI/Flask backend exposes endpoints to:  
     - Fetch recent anomalies by user, risk level, or time window.  
     - Inspect transaction details, feature values, and model outputs.[web:135][web:141]  
   - A simple dashboard visualizes:  
     - Real-time stream of anomalies.  
     - Risk distribution by geography, merchant, and time-of-day.  
     - Historical trends in anomaly rates and false positives.  

5. **Explainability & feedback loop**  
   - Use model-specific techniques (e.g., feature importances, SHAP) to show which features contributed most to the anomaly score.[web:135][web:138]  
   - Allow reviewers to label each alert as **true fraud** or **false positive**, and store this feedback.[web:140][web:149]  
   - Regularly retrain models and adjust thresholds using this labelled data.  

6. **Monitoring & reliability**  
   - Track key metrics:  
     - End-to-end latency from transaction ingestion to alert.  
     - Throughput (transactions/sec), Kafka lag, Spark batch duration.[web:136][web:139]  
     - Model performance metrics where labels exist (precision, recall, false positive rate).[web:140]  
   - Surface metrics in Prometheus/Grafana or similar tools for production-style observability.  

---

## 📁 Suggested Repository Structure

