# Data Pipeline with PySpark, Kafka, and Redshift

This repository demonstrates building a real-time data pipeline using **PySpark**, **Apache Kafka**, and **Amazon Redshift**. The project includes two key integrations:

1. **PySpark with Kafka** for real-time data streaming.
2. **PySpark with Amazon Redshift** for storing and querying processed data.

## Project Overview

### PySpark + Kafka Integration
- **Kafka Producer Setup**: Uses the `confluent-kafka` library to produce JSON-serialized messages and send them to Kafka topics.
- **Data Processing with PySpark**: Consumes data from Kafka streams using PySpark and processes it for real-time analysis.
- **Use Cases**: Ideal for applications requiring low-latency processing like real-time monitoring, fraud detection, and live data analytics.

### PySpark + Redshift Integration
- **Connecting to Redshift**: Uses a JDBC driver to connect PySpark with Amazon Redshift for data ingestion and querying.
- **Data Transformation**: Performs data transformations using PySpark and writes the processed data into Redshift tables.
- **Use Cases**: Useful for scenarios where batch processing and data warehousing are necessary, such as generating reports and conducting large-scale analytics.

## Prerequisites

- Python 3.7+
- Apache Kafka (locally or using Docker)
- Amazon Redshift
- PySpark
- `confluent-kafka` library
- Redshift JDBC Driver (`redshift-jdbc42-2.0.0.4.jar`)

## Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/1varma/Build-a-Data-Pipeline-using-kafka-and-Redshift.git
   cd Build-a-Data-Pipeline-using-kafka-and-Redshift
   ```

2. **Install Dependencies**:
   Install the required Python packages:
   ```bash
   pip install confluent-kafka pandas pyspark
   ```

3. **Set Up Kafka**:
   Ensure that Kafka is running on your local machine or Docker with a bootstrap server at `localhost:9092`.

4. **Download Redshift JDBC Driver**:
   Download the `redshift-jdbc42-2.0.0.4.jar` and place it in the project directory.

## Usage

### 1. Kafka Integration with PySpark

**Run the Kafka Producer**: The `KafkaProducer` sends JSON-serialized data to a Kafka topic.
```python
from json import dumps
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda x: dumps(x).encode('utf-8')
)

# Send data to Kafka
producer.send('your_topic', value={'key': 'value'})
```

**Consume Data with PySpark**: Use PySpark to read data from Kafka and process it:
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName('KafkaPySpark') \
    .getOrCreate()

# Read from Kafka
df = spark.readStream \
    .format('kafka') \
    .option('kafka.bootstrap.servers', 'localhost:9092') \
    .option('subscribe', 'your_topic') \
    .load()

df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)") \
  .writeStream \
  .format('console') \
  .start() \
  .awaitTermination()
```

### 2. Redshift Integration with PySpark

**Set Up Spark Session with Redshift JDBC Driver**:
```python
from pyspark.sql import SparkSession

# Specify the path to the JDBC driver
jar_path = 'redshift-jdbc42-2.0.0.4.jar'

# Create Spark session with Redshift support
spark = SparkSession.builder \
    .appName('integration_pyspark_redshift') \
    .config('spark.jars', jar_path) \
    .config('spark.driver.extraClassPath', jar_path) \
    .getOrCreate()
```

**Write Data to Redshift**:
```python
# Define connection properties
url = 'jdbc:redshift://<redshift-cluster-endpoint>:5439/<database>'
properties = {
    'user': 'your_username',
    'password': 'your_password'
}

# Write a PySpark DataFrame to Redshift
df.write \
    .format('jdbc') \
    .option('url', url) \
    .option('dbtable', 'your_table_name') \
    .option('user', properties['user']) \
    .option('password', properties['password']) \
    .save()
```

**Read Data from Redshift**:
```python
# Read data from a Redshift table into a PySpark DataFrame
df = spark.read \
    .format('jdbc') \
    .option('url', url) \
    .option('dbtable', 'your_table_name') \
    .option('user', properties['user']) \
    .option('password', properties['password']) \
    .load()
```

## Notebook Guides

- **`pyspark_kafka.ipynb`**: A Jupyter notebook that demonstrates integrating PySpark with Kafka for real-time data processing.
- **`pyspark_redshift.ipynb`**: A Jupyter notebook that showcases how to connect PySpark with Amazon Redshift for reading and writing data.

## Contributing

Feel free to open issues or submit pull requests if you have suggestions or improvements. Contributions are always welcome!

## Contact

For more information, reach out via [LinkedIn](https://www.linkedin.com/in/ashishvarmajuttu).
