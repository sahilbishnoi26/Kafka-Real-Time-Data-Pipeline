# Kafka-Real-Time-Data-Pipeline

This project builds a real-time stock market data processing pipeline using **Kafka** and **AWS services**. It simulates stock market data, ingests it into Kafka, consumes and stores it in **Amazon S3**, catalogs it using **AWS Glue**, and analyzes it with **Amazon Athena**.

## Architecture  
1. **Data Simulation**: Reads historical stock market data and streams it in real time.  
2. **Kafka Producer**: Pushes simulated stock market events to a **Kafka** cluster.  
3. **Kafka Consumer**: Reads data from Kafka and stores it in **Amazon S3**.  
4. **AWS Glue**: Crawls S3 data and builds a **Glue Catalog** for structured querying.  
5. **Amazon Athena**: Queries real-time stock market data using **SQL**.  

## Tech Stack  
- **Kafka** (Event Streaming)  
- **Python** (Data Simulation, Producer, Consumer)  
- **Amazon S3** (Storage)  
- **AWS Glue** (Schema Extraction)  
- **Amazon Athena** (Querying)  
- **EC2** (Kafka Hosting)  

## Setup Instructions  

### 1. Prerequisites  
- **AWS Account** (for S3, Glue, Athena, EC2)  
- **Kafka Installed on EC2**  
- **Python 3.5+**  
- **Jupyter Notebook** (for interactive execution)  

### 2. Install Dependencies  
```bash
pip install kafka-python pandas s3fs boto3
```

### 3. Kafka Setup on EC2  
- Launch an **EC2 instance** (Amazon Linux 2)  
- Install **Kafka** and **Zookeeper**  
- Configure **Security Groups** to allow external access  

### 4. Start Kafka Services  
```bash
# Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka Broker
bin/kafka-server-start.sh config/server.properties
```

### 5. Create a Kafka Topic  
```bash
bin/kafka-topics.sh --create --topic stock_market --bootstrap-server <EC2-Public-IP>:9092 --partitions 1 --replication-factor 1
```

### 6. Run Producer (Simulated Stock Market Data)  
```python
from kafka import KafkaProducer
import json
import pandas as pd
import time

producer = KafkaProducer(
    bootstrap_servers="<EC2-Public-IP>:9092",
    value_serializer=lambda v: json.dumps(v).encode("utf-8"),
)

df = pd.read_csv("stock_data.csv")

while True:
    row = df.sample(1).to_dict(orient="records")[0]
    producer.send("stock_market", value=row)
    time.sleep(1)
```

### 7. Run Consumer (Save to S3)  
```python
from kafka import KafkaConsumer
import json
import s3fs

consumer = KafkaConsumer(
    "stock_market",
    bootstrap_servers="<EC2-Public-IP>:9092",
    value_deserializer=lambda v: json.loads(v.decode("utf-8")),
)

s3 = s3fs.S3FileSystem()

for message in consumer:
    data = message.value
    file_name = f"stock_market_{int(time.time())}.json"
    with s3.open(f"s3://<BUCKET_NAME>/{file_name}", "w") as f:
        f.write(json.dumps(data))
```

### 8. AWS Glue Setup  
- **Create a Glue Crawler** to detect schema from S3.  
- **Run the crawler** to populate the **Glue Catalog**.  

### 9. Query Data in Athena  
```sql
SELECT * FROM stock_market_data LIMIT 10;
```

## Next Steps  
- **Enhance the architecture** with distributed Kafka clusters.  
- **Stream real stock market data** using an API.  
- **Store data in a database** like Snowflake or Redshift.  

