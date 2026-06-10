# LogPulse: Cloud-Based Log Analytics Platform

LogPulse is a production-style log analytics platform with fault-tolerant ingestion, Elasticsearch indexing/search, Python alerting, and a React operations dashboard.

## Architecture

```text
Applications -> Spring Boot Collector -> Kafka -> Spring Boot Indexer -> Elasticsearch
                                                -> Python Alert Engine -> Slack/Webhook
                                                -> React Dashboard -> Search API
```

## Features

- Java Spring Boot ingestion API with validation, structured logging, rate limiting, and Kafka publishing.
- Kafka consumer indexing into Elasticsearch with retry-safe processing.
- Search API for keyword search, service filtering, severity filtering, and time windows.
- Python alert engine that evaluates Elasticsearch-backed rules and sends webhook notifications.
- React dashboard for log search, service health, and error trend visualization.
- Docker Compose for local production-like deployment.
- Unit tests for backend services, alert rules, and frontend components.

## Repository Structure

```text
backend/         Spring Boot collector, indexer, and search API
frontend/        React dashboard
alerting/        Python alerting service
config/          Elasticsearch index template, sample env files, nginx config
scripts/         Developer and deployment scripts
docs/            API, architecture, security, and operations docs
.github/         CI workflow
```

## Quick Start

### Prerequisites

- Docker 24+
- Docker Compose v2+
- Java 21
- Maven 3.9+
- Node.js 20+
- Python 3.11+

### Run the full stack

```bash
cp config/env.example .env
docker compose up --build
```

Services:

| Service | URL |
|---|---|
| React dashboard | http://localhost:8080 |
| Backend API | http://localhost:8081 |
| Elasticsearch | http://localhost:9200 |
| Kafka broker | localhost:9092 |

### Send a log event

```bash
curl -X POST http://localhost:8081/api/v1/logs \
  -H 'Content-Type: application/json' \
  -H 'X-API-Key: dev-api-key' \
  -d '{
    "service":"billing-service",
    "environment":"prod",
    "level":"ERROR",
    "message":"Payment provider timeout",
    "traceId":"trace-123",
    "metadata":{"orderId":"A100"}
  }'
```

### Search logs

```bash
curl 'http://localhost:8081/api/v1/logs/search?service=billing-service&level=ERROR&from=now-24h&size=20' \
  -H 'X-API-Key: dev-api-key'
```

## Environment Variables

| Name | Default | Description |
|---|---:|---|
| `LOGPULSE_API_KEY` | `dev-api-key` | Required API key for backend requests. |
| `KAFKA_BOOTSTRAP_SERVERS` | `localhost:9092` | Kafka bootstrap server list. |
| `LOGPULSE_LOG_TOPIC` | `logpulse.logs` | Kafka topic for log events. |
| `ELASTICSEARCH_URIS` | `http://localhost:9200` | Elasticsearch endpoint. |
| `ELASTICSEARCH_INDEX` | `logpulse-logs` | Primary log index alias/name. |
| `ALERT_WEBHOOK_URL` | empty | Optional alert destination webhook. |
| `ALERT_POLL_SECONDS` | `30` | Alert evaluation interval. |

## Development

Backend:

```bash
cd backend
mvn test
mvn spring-boot:run
```

Frontend:

```bash
cd frontend
npm install
npm run test
npm run build
npm run dev
```

Alerting:

```bash
cd alerting
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
pytest
python -m logpulse_alerting.main
```

## Security Practices

- All API endpoints require `X-API-Key` except health checks.
- API key is supplied through environment variables, never hard-coded for real deployments.
- Inputs are validated using Jakarta Bean Validation.
- CORS is restricted through configuration.
- Elasticsearch queries are built with typed query builders to reduce injection risk.
- Containers run with minimal images and explicit ports.

## License

MIT
