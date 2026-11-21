# AWS Containers Retail Sample - Checkout Service

| Language | Persistence |
| -------- | ----------- |
| Node     | Redis       |

This service provides an API for storing customer data during the checkout process. Data is stored in Redis.

## Configuration

The following environment variables are available for configuring the service:

| Name                                    | Description                                                              | Default     |
| --------------------------------------- | ------------------------------------------------------------------------ | ----------- |
| `PORT`                                  | The port which the server will listen on                                 | `8080`      |
| `RETAIL_CHECKOUT_PERSISTENCE_PROVIDER`  | The persistence provider to use, can be `in-memory` or `redis`.          | `in-memory` |
| `RETAIL_CHECKOUT_PERSISTENCE_REDIS_URL` | The endpoint of the Redis server used to store state, for example `redis://localhost:6379`. | `""`        |
| `RETAIL_CHECKOUT_ENDPOINTS_ORDERS`      | The endpoint of the orders API. If empty uses a mock implementation      | `""`        |
| `RETAIL_CHECKOUT_SHIPPING_NAME_PREFIX`  | A string prefix that can be applied to the names of the shipping options | `""`        |

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

**Short version:** `yarn install`, then `yarn serve`. It keeps data in memory, so **you do not need Redis** to try it.

### 1. What you need

- **Node.js 20 or newer** (`node -v`). NestJS 11 does not run on older versions. An older note said Node 16; that is wrong.
- Yarn 1.x. If `yarn` is missing: `npm install -g yarn`.

### 2. Start it

```
cd src/checkout
yarn install
yarn serve
```

There is **no** `yarn start` script. Use `yarn serve`, or `yarn serve:dev` to restart automatically when you edit a file.

### 3. Check that it works

```
curl localhost:8080/health
# {"status":"ok","info":{"chaos":{"status":"up"}}, ...}
```

Orders are mocked unless you set `RETAIL_CHECKOUT_ENDPOINTS_ORDERS`.

### 4. Run with the real Redis (optional)

```
docker run -d --name checkout-redis -p 6379:6379 redis:7

RETAIL_CHECKOUT_PERSISTENCE_PROVIDER=redis \
RETAIL_CHECKOUT_PERSISTENCE_REDIS_URL=redis://localhost:6379 \
yarn serve
```

Clean up with `docker rm -f checkout-redis`.

### 5. Connect it to the real orders service (optional)

Start `orders` first, then:

```
RETAIL_CHECKOUT_ENDPOINTS_ORDERS=http://localhost:8083 PORT=8084 yarn serve
```

### 6. Docker

The `Dockerfile` for this service is written in Phase 2 (issue #8). Until then, use the steps above.

### Common problems

| You see | Cause and fix |
| ------- | ------------- |
| `The engine "node" is incompatible with this module. Expected version ">=20"` | Node is older than 20. Install Node 20 or newer. |
| `yarn: command not found` | Run `npm install -g yarn`. |
| `error Command "start" not found` | There is no start script. Use `yarn serve`. |
| `EADDRINUSE :::8080` | Port 8080 is busy. Use `PORT=8084 yarn serve`. |

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
