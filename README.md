# Real-Time CDC Data Pipeline with Debezium & Kafka

A production-ready Change Data Capture (CDC) pipeline that streams database changes from PostgreSQL to Kafka in real-time using Debezium. This project demonstrates modern data engineering practices including event streaming, database replication, and real-time data processing.

## 🏗️ Architecture
```
PostgreSQL (WAL) → Debezium Connect → Kafka → [Your Consumer Applications]
                         ↓
                   Schema Registry
```

This pipeline captures row-level changes (INSERT, UPDATE, DELETE) from PostgreSQL and publishes them as events to Kafka topics, enabling real-time data integration and event-driven architectures.

## 🚀 Features

- **Change Data Capture**: Real-time streaming of database changes using Debezium
- **Event Streaming**: Apache Kafka for reliable, scalable message streaming
- **Schema Management**: Confluent Schema Registry for schema evolution
- **Monitoring UI**: Kafdrop for Kafka topic visualization and Debezium UI for connector management
- **Logical Replication**: PostgreSQL configured with Write-Ahead Logging (WAL)
- **Docker Compose**: Fully containerized setup for easy deployment

## 📋 Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- 4GB+ RAM allocated to Docker
- Ports available: 5432, 8080, 8081, 8083, 9000, 9092

## 🛠️ Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Database | PostgreSQL 13 | Source database with logical replication |
| CDC | Debezium 2.1 | Change data capture connector |
| Streaming | Apache Kafka | Distributed event streaming platform |
| Coordination | Apache Zookeeper | Kafka cluster coordination |
| Schema Registry | Confluent Schema Registry | Schema versioning and validation |
| Monitoring | Kafdrop | Kafka topic browser and monitoring |
| UI | Debezium UI | Connector management interface |

## 🚦 Getting Started

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/cdc-kafka-pipeline.git
cd cdc-kafka-pipeline
```

### 2. Start the Infrastructure
```bash
docker-compose up -d
```

Wait for all services to be healthy (~30-60 seconds):
```bash
docker-compose ps
```

### 3. Configure Debezium Connector

Create a PostgreSQL CDC connector:
```bash
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" \
  localhost:8083/connectors/ -d '{
  "name": "postgres-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "docker",
    "database.password": "docker",
    "database.dbname": "exampledb",
    "database.server.name": "dbserver1",
    "table.include.list": "public.*",
    "plugin.name": "pgoutput"
  }
}'
```

### 4. Test the Pipeline

Create a test table and insert data:
```bash
docker exec -it postgres psql -U docker -d exampledb

CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW()
);

INSERT INTO customers (name, email) VALUES 
  ('John Doe', 'john@example.com'),
  ('Jane Smith', 'jane@example.com');
```

### 5. View the Changes

Open Kafdrop at http://localhost:9000 to see the CDC events in the `dbserver1.public.customers` topic.

## 📊 Management Interfaces

| Service | URL | Purpose |
|---------|-----|---------|
| Debezium UI | http://localhost:8080 | Manage CDC connectors |
| Kafdrop | http://localhost:9000 | Browse Kafka topics and messages |
| Schema Registry | http://localhost:8081 | View registered schemas |
| Debezium Connect API | http://localhost:8083 | Connector REST API |

## 🔧 Configuration

### PostgreSQL Settings

The database is configured with:
- `wal_level=logical` - Enables logical replication
- Default user: `docker` / password: `docker`
- Database: `exampledb`

### Kafka Settings

- Single broker setup (suitable for development)
- Replication factor: 1
- Advertised listener: `kafka:9092`

### Network

All services communicate through the `cdc-network` bridge network.

## 📁 Project Structure
```
.
├── docker-compose.yml      # Infrastructure definition
├── README.md              # This file
├── connectors/            # Debezium connector configs (optional)
└── consumers/             # Example consumer applications (optional)
```

## 🔍 Common Operations

### View Connector Status
```bash
curl http://localhost:8083/connectors/postgres-connector/status
```

### List Kafka Topics
```bash
docker exec -it kafka kafka-topics --list --bootstrap-server localhost:9092
```

### Consume Messages
```bash
docker exec -it kafka kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic dbserver1.public.customers \
  --from-beginning
```

### Stop All Services
```bash
docker-compose down
```

### Clean Up (including volumes)
```bash
docker-compose down -v
```

## 🎯 Use Cases

- Real-time data replication
- Event-driven microservices
- Data lake ingestion
- Cache invalidation
- Audit logging
- Real-time analytics
- Data synchronization across systems

## 🐛 Troubleshooting

### Connector fails to start

Check PostgreSQL is configured for logical replication:
```bash
docker exec postgres psql -U docker -d exampledb -c "SHOW wal_level;"
```

### Can't connect to Kafka

Verify Kafka is running and accessible:
```bash
docker logs kafka
```

### No CDC events appearing

1. Check connector status in Debezium UI
2. Verify table is in the whitelist
3. Ensure PostgreSQL user has replication permissions

## 🚀 Next Steps

To extend this project:

1. **Add Consumer Applications**: Build services that consume CDC events
2. **Data Transformations**: Use Kafka Streams for real-time processing
3. **Multiple Databases**: Add MySQL, MongoDB, or other source connectors
4. **Sink Connectors**: Stream data to Elasticsearch, S3, or data warehouses
5. **Monitoring**: Add Prometheus and Grafana for metrics
6. **Production Hardening**: Configure authentication, SSL, and multi-broker setup

## 📚 Learning Resources

- [Debezium Documentation](https://debezium.io/documentation/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [PostgreSQL Logical Replication](https://www.postgresql.org/docs/current/logical-replication.html)

## 📝 License

This project is open source and available under the MIT License.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the issues page.

---

**Built with** ❤️ **as a Data Engineering Portfolio Project**
