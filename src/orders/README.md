# AWS Containers Retail Sample - Orders Service

| Language | Persistence |
| -------- | ----------- |
| Java     | PostgreSQL  |

This service provides an API for storing orders. Data is stored in PostgreSQL.

## Configuration

The following environment variables are available for configuring the service:

| Name                                          | Description                                                                             | Default       |
| --------------------------------------------- | --------------------------------------------------------------------------------------- | ------------- |
| `PORT`                                        | The port which the server will listen on                                                | `8080`        |
| `RETAIL_ORDERS_PERSISTENCE_PROVIDER`          | The persistence provider to use, can be `in-memory` or `postgres`.                      | `in-memory`   |
| `RETAIL_ORDERS_PERSISTENCE_ENDPOINT`          | The postgres database endpoint.                                                         | `""`          |
| `RETAIL_ORDERS_PERSISTENCE_NAME`              | The postgres database name                                                              | `""`          |
| `RETAIL_ORDERS_PERSISTENCE_USERNAME`          | Username to authenticate with postgres database.                                        | `""`          |
| `RETAIL_ORDERS_PERSISTENCE_PASSWORD`          | Password to authenticate with postgres database.                                        | `""`          |
| `RETAIL_ORDERS_MESSAGING_PROVIDER`            | The messaging provider to use to publish events. Can be `in-memory`, `sqs`, `rabbitmq`. | `"in-memory"` |
| `RETAIL_ORDERS_MESSAGING_SQS_TOPIC`           | The name of the SQS topic to publish events to (SQS messaging provider)                 | `""`          |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_ADDRESSES`  | Endpoints for RabbitMQ messaging provider, format `host:port`                           | `""`          |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_USERNAME`   | Username for RabbitMQ messaging provider                                                | `""`          |
| `RETAIL_ORDERS_MESSAGING_RABBITMQ_PASSWORD`   | Password for RabbitMQ messaging provider                                                | `""`          |

## Endpoints

Several "utility" endpoints are provided with useful functionality for various scenarios:

| Method   | Name                     | Description                                                                        |
| -------- | ------------------------ | ---------------------------------------------------------------------------------- |
| `POST`   | `/chaos/status/{code}`   | All HTTP requests to API paths will return the given HTTP status code              |
| `DELETE` | `/chaos/status`          | Disables the HTTP status response above                                            |
| `POST`   | `/chaos/latency/{delay}` | All HTTP requests to API paths will have the specified delay added in milliseconds |
| `DELETE` | `/chaos/latency`         | Disables the HTTP response latency above                                           |
| `POST`   | `/chaos/health`          | Causes all health check requests to fail                                           |
| `DELETE` | `/chaos/health`          | Returns the health check to its default behavior                                   |

## Run it locally

**Short version:** `./mvnw spring-boot:run`. It keeps orders in memory, so **you do not need PostgreSQL or RabbitMQ** to try it.

### 1. What you need

- Java 21 (`java -version` must say 21)
- Internet on the first run, to download dependencies

### 2. Start it

```
cd src/orders
./mvnw spring-boot:run
```

### 3. Check that it works

```
curl localhost:8080/actuator/health        # {"status":"UP"}
curl localhost:8080/orders                 # []   (no orders yet)
```

In-memory orders are lost when you stop the service.

### 4. Run with the real PostgreSQL database (optional)

```
docker run -d --name orders-postgres -p 5432:5432 \
  -e POSTGRES_DB=orders -e POSTGRES_USER=orders_user -e POSTGRES_PASSWORD=testing postgres:16

RETAIL_ORDERS_PERSISTENCE_PROVIDER=postgres \
RETAIL_ORDERS_PERSISTENCE_ENDPOINT=localhost:5432 \
RETAIL_ORDERS_PERSISTENCE_NAME=orders \
RETAIL_ORDERS_PERSISTENCE_USERNAME=orders_user \
RETAIL_ORDERS_PERSISTENCE_PASSWORD=testing \
./mvnw spring-boot:run
```

The log says `Using postgres database`, and Flyway creates the tables by itself. All four `RETAIL_ORDERS_PERSISTENCE_*` connection values must be set. Clean up with `docker rm -f orders-postgres`.

> **Note the variable name:** it is `RETAIL_ORDERS_PERSISTENCE_PROVIDER`. An older version of this file said `RETAIL_CHECKOUT_PERSISTENCE_PROVIDER`, which is wrong.

### 5. RabbitMQ (optional)

Order events stay in memory unless you set `RETAIL_ORDERS_MESSAGING_PROVIDER=rabbitmq` together with the `RETAIL_ORDERS_MESSAGING_RABBITMQ_*` values from the table above. You only need this when you test the full event flow.

### 6. Docker

The `Dockerfile` for this service is written in Phase 2 (issue #6). Until then, use the steps above.

### Common problems

| You see | Cause and fix |
| ------- | ------------- |
| `release version 21 not supported` or `invalid target release` | Your Java is older than 21. Run `java -version` and install Java 21. |
| First start takes 1 to 3 minutes | Maven is downloading dependencies. Later starts take a few seconds. |
| `Port 8080 was already in use` | Something else uses 8080. Start with another port: `PORT=8085 ./mvnw spring-boot:run` |
| `./mvnw: Permission denied` | Run `chmod +x mvnw` once. |

### Running several services on one machine

Every service listens on port **8080** by default, so two of them clash. Give each its own port with `PORT`:

| Service | Suggested port |
| ------- | -------------- |
| ui | 8080 |
| catalog | 8081 |
| cart | 8082 |
| orders | 8083 |
| checkout | 8084 |

Example: `PORT=8082 ./mvnw spring-boot:run`. To start the whole shop with one command, use the `docker-compose.yml` in the repository root (see the main README).
