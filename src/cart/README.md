# AWS Containers Retail Sample - Cart Service

| Language | Persistence     |
| -------- | --------------- |
| Java     | DynamoDB Local (a container on your machine) |

This service provides an API for storing customer shopping carts.

> **Nothing here runs on AWS.** In our setup the data is stored in **DynamoDB Local**, a free container that runs on your own machine or server and speaks the same protocol as Amazon DynamoDB. No AWS account is needed. The `AWS_*` variables are only dummy values that the client library insists on.

## Configuration

The following environment variables are available for configuring the service:

| Name                                            | Description                                                        | Default     |
| ----------------------------------------------- | ------------------------------------------------------------------ | ----------- |
| `PORT`                                          | The port which the server will listen on                           | `8080`      |
| `RETAIL_CART_PERSISTENCE_PROVIDER`              | The persistence provider to use, can be `in-memory` or `dynamodb`. | `in-memory` |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_TABLE_NAME`   | The name of the Amazon DynamoDB table used for persistence         | `Items`     |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_ENDPOINT`     | The Amazon DynamoDB endpoint to use                                | `""`        |
| `RETAIL_CART_PERSISTENCE_DYNAMODB_CREATE_TABLE` | Enable to automatically create the Amazon DynamoDB table required  | `false`     |

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

**Short version:** `./mvnw spring-boot:run`. It stores carts in memory, so **you do not need DynamoDB** to try it.

### 1. What you need

- Java 21 (`java -version` must say 21)
- Internet on the first run, to download dependencies

### 2. Start it

```
cd src/cart
./mvnw spring-boot:run
```

### 3. Check that it works

```
curl localhost:8080/actuator/health                 # {"status":"UP"}
curl localhost:8080/carts/demo                      # {"customerId":"demo","items":[]}

# add an item to the cart of customer "demo"
curl -X POST localhost:8080/carts/demo/items \
  -H 'content-type: application/json' \
  -d '{"itemId":"abc","quantity":1,"unitPrice":10}'

curl localhost:8080/carts/demo                      # the item is there
```

In-memory carts are lost when you stop the service.

### 4. Run with DynamoDB Local (optional)

DynamoDB Local is a container that behaves like Amazon DynamoDB. You do **not** need an AWS account.

```
docker run -d --name cart-dynamodb -p 8000:8000 amazon/dynamodb-local

AWS_ACCESS_KEY_ID=dummy AWS_SECRET_ACCESS_KEY=dummy \
RETAIL_CART_PERSISTENCE_PROVIDER=dynamodb \
RETAIL_CART_PERSISTENCE_DYNAMODB_ENDPOINT=http://localhost:8000 \
RETAIL_CART_PERSISTENCE_DYNAMODB_CREATE_TABLE=true \
./mvnw spring-boot:run
```

The two `AWS_*` values can be any text, but they **must be set**. Without them the service stops with `Unable to load credentials from any of the providers in the chain`. `CREATE_TABLE=true` creates the `Items` table for you. Clean up with `docker rm -f cart-dynamodb`.

### 5. Docker

The `Dockerfile` for this service is written in Phase 2 (issue #5). Until then, use the steps above.

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
