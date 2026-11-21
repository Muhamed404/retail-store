# AWS Containers Retail Sample - UI Service

| Language | Persistence |
| -------- | ----------- |
| Java     | N/A         |

This service provides the frontend for the retail store, serving the HTML UI and aggregating calls to the backend API components.

## Configuration

The following environment variables are available for configuring the service:

| Name                              | Description                                                                                            | Default                 |
| --------------------------------- | ------------------------------------------------------------------------------------------------------ | ----------------------- |
| `PORT`                            | The port which the server will listen on                                                               | `8080`                  |
| `RETAIL_UI_THEME`                 | Name of the theme for the UI, valid values are `default`, `green`, `orange`, `teal`                    | `"default"`             |
| `RETAIL_UI_DISABLE_DEMO_WARNINGS` | Disable the UI messages warning about demonstration content                                            | `false`                 |
| `RETAIL_UI_PRODUCT_IMAGES_PATH`   | Overrides the location where the sample product images are sourced from to use the specified file path | ``                      |
| `RETAIL_UI_ENDPOINTS_CATALOG`     | The endpoint of the catalog API. If set to `false` uses a mock implementation                          | `false`                 |
| `RETAIL_UI_ENDPOINTS_CARTS`       | The endpoint of the carts API. If set to `false` uses a mock implementation                            | `false`                 |
| `RETAIL_UI_ENDPOINTS_ORDERS`      | The endpoint of the orders API. If set to `false` uses a mock implementation                           | `false`                 |
| `RETAIL_UI_ENDPOINTS_CHECKOUT`    | The endpoint of the checkout API. If set to `false` uses a mock implementation                         | `false`                 |
| `RETAIL_UI_CHAT_ENABLED`          | Enable the chat bot UI                                                                                 | `false`                 |
| `RETAIL_UI_CHAT_PROVIDER`         | The chat provider to use, value values are `bedrock`, `openai`, `mock`                                 | `""`                    |
| `RETAIL_UI_CHAT_MODEL`            | The chat model to use, depends on the provider.                                                        | `""`                    |
| `RETAIL_UI_CHAT_TEMPERATURE`      | Model temperature                                                                                      | `0.6`                   |
| `RETAIL_UI_CHAT_MAX_TOKENS`       | Model maximum response tokens                                                                          | `300`                   |
| `RETAIL_UI_CHAT_PROMPT`           | Model system prompt                                                                                    | `(see source)`          |
| `RETAIL_UI_CHAT_BEDROCK_REGION`   | Amazon Bedrock region                                                                                  | `""`                    |
| `RETAIL_UI_CHAT_OPENAI_BASE_URL`  | Base URL for OpenAI endpoint                                                                           | `http://localhost:8888` |
| `RETAIL_UI_CHAT_OPENAI_API_KEY`   | API key for OpenAI endpoint                                                                            | `""`                    |

## Endpoints

Several "utility" endpoints are provided with useful functionality for various scenarios:

| Method | Name                           | Description                                                                 |
| ------ | ------------------------------ | --------------------------------------------------------------------------- |
| `GET`  | `/utility/status/{code}`       | Returns HTTP response with given HTTP status code                           |
| `GET`  | `/utility/headers`             | Print the HTTP headers of the inbound request                               |
| `GET`  | `/utility/panic`               | Shutdown the application with an error code                                 |
| `POST` | `/utility/echo`                | Write back the POST payload sent                                            |
| `POST` | `/utility/store`               | Write the payload to a file and return a hash                               |
| `GET`  | `/utility/store/{hash}`        | Return the payload from the file system previously written                  |
| `GET`  | `/utility/stress/{iterations}` | Stress the CPU with the number of iterations increasing the CPU consumption |

## Run it locally

**Short version:** `./mvnw spring-boot:run`, then open <http://localhost:8080>. You do not need any other service or database.

### 1. What you need

- Java 21 (`java -version` must say 21)
- Internet on the first run, to download dependencies

### 2. Start it

```
cd src/ui
./mvnw spring-boot:run
```

Wait for the line `Started UiApplication`.

### 3. Check that it works

```
curl localhost:8080/actuator/health        # {"status":"UP"}
```

Open <http://localhost:8080> in a browser. You see the **Demo Store** with sample products. By default the UI uses **built-in mock data** for the catalog, cart, orders and checkout, so it works alone.

### 4. Connect it to the real services (optional)

Start the other services first (see their READMEs), then tell the UI where they are:

```
RETAIL_UI_ENDPOINTS_CATALOG=http://localhost:8081 \
RETAIL_UI_ENDPOINTS_CARTS=http://localhost:8082 \
RETAIL_UI_ENDPOINTS_ORDERS=http://localhost:8083 \
RETAIL_UI_ENDPOINTS_CHECKOUT=http://localhost:8084 \
./mvnw spring-boot:run
```

Use the ports from the table below.

### 5. Docker

The `Dockerfile` for this service is written in Phase 2 (issue #4). Until then, use the steps above.

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
