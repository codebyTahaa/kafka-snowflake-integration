# 🧭 Kafka to Snowflake Integration Project

This project demonstrates how to stream data from **Apache Kafka** into **Snowflake** using **Snowflake Kafka Connector**. It's a practical example of building real-time data pipelines suitable for analytics, reporting, or warehousing.

---

## 📌 Project Goals

- ✅ Stream real-time data from Kafka topics into Snowflake tables  
- ✅ Use Snowflake Kafka Connector for automatic ingestion  
- ✅ Handle JSON-formatted messages  
- ✅ Create Snowflake stages, pipes, and required resources  
- ✅ Demonstrate end-to-end data flow for analytics use cases  

---

## 🧰 Tech Stack

| Component           | Technology               |
|---------------------|--------------------------|
| Stream Processor    | Apache Kafka             |
| Messaging Format    | JSON                     |
| Data Warehouse      | Snowflake                |
| Connector Tool      | Snowflake Kafka Connector |
| Language            | Java / Python (optional) |
| Cloud Provider      | AWS (for storage & key delivery) |

---

## 🗂️ Folder Structure

```
kafka-to-snowflake/
│
├── docker-compose.yml        # Kafka + Zookeeper (local setup)
├── producer.py               # (Optional) Python Kafka producer
├── snowflake/                
│   ├── create_stage.sql      # Create external stage in Snowflake
│   ├── create_pipe.sql       # Define Snowpipe for auto-ingestion
│   └── table_schema.sql      # Define target Snowflake table
├── config/
│   └── kafka-connector.json  # Kafka → Snowflake connector config
├── README.md                 # Documentation
```

---

## ⚙️ Setup Instructions

### 1. Spin up Kafka locally (optional)

```bash
docker-compose up -d
```

This launches Kafka, Zookeeper, and the Kafka UI locally.

### 2. Create Snowflake objects

Execute these SQL scripts inside your Snowflake worksheet:

```sql
-- Step 1: Create target table
CREATE OR REPLACE TABLE your_db.your_schema.kafka_stream_data (
    id STRING,
    event_time TIMESTAMP,
    payload VARIANT
);

-- Step 2: Create stage (S3 or internal)
CREATE OR REPLACE STAGE your_stage
  URL='s3://your-bucket-name/kafka-data/'
  STORAGE_INTEGRATION = your_integration;

-- Step 3: Create pipe for auto-ingestion
CREATE OR REPLACE PIPE your_pipe
  AUTO_INGEST = TRUE
  AS
  COPY INTO your_db.your_schema.kafka_stream_data
  FROM @your_stage
  FILE_FORMAT = (TYPE = JSON);
```

### 3. Deploy the Kafka Connector

Configure the `kafka-connector.json` file with your Snowflake account info:

```json
{
  "name": "snowflake-connector",
  "config": {
    "topics": "your-kafka-topic",
    "connector.class": "com.snowflake.kafka.connector.SnowflakeSinkConnector",
    "tasks.max": "1",
    "snowflake.url.name": "your_account.snowflakecomputing.com:443",
    "snowflake.user.name": "YOUR_USER",
    "snowflake.private.key": "YOUR_PRIVATE_KEY",
    "snowflake.database.name": "YOUR_DB",
    "snowflake.schema.name": "YOUR_SCHEMA",
    "snowflake.table.name": "kafka_stream_data",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false"
  }
}
```

Then deploy it via Kafka Connect REST API:

```bash
curl -X POST -H "Content-Type: application/json" \
     --data @config/kafka-connector.json \
     http://localhost:8083/connectors
```

---

## 🚀 Sample Kafka Producer (Optional)

You can use a Python producer to send sample JSON data to Kafka:

```python
from kafka import KafkaProducer
import json
import time

producer = KafkaProducer(bootstrap_servers='localhost:9092',
                         value_serializer=lambda v: json.dumps(v).encode('utf-8'))

while True:
    data = {"id": "abc123", "event_time": "2025-08-02T12:00:00Z", "payload": {"type": "click", "source": "web"}}
    producer.send("your-kafka-topic", data)
    print("Message sent!")
    time.sleep(5)
```

---

## 📈 End-to-End Flow

```
Kafka Producer ───► Kafka Topic ───► Snowflake Kafka Connector ───► Snowflake Table
```

---

## ✅ Use Cases

- Real-time event processing (clickstreams, logs)
- IoT sensor data ingestion
- Application telemetry pipelines
- Streaming ETL workflows

---

## 🙋‍♂️ Author

**Muhammad Taha**  
Cloud & Data Engineer | Kafka | Snowflake  
📍 Saylani Mass IT Training  
🔗 www.linkedin.com/in/muhammad-taha-023a89287

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).
