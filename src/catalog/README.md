# AWS Containers Retail Sample - Catalog Service

| Language | Persistence |
| -------- | ----------- |
| Go       | MySQL       |

This service provides an API for retrieving product catalog information. Data is stored in a MySQL database.

## Configuration

The following environment variables are available for configuring the service:

| Name                                       | Description                                                     | Default        |
| ------------------------------------------ | --------------------------------------------------------------- | -------------- |
| PORT                                       | The port which the server will listen on                        | `8080`         |
| RETAIL_CATALOG_PERSISTENCE_PROVIDER        | The persistence provider to use, can be `in-memory` or `mysql`. | `in-memory`    |
| RETAIL_CATALOG_PERSISTENCE_ENDPOINT        | Database endpoint URL                                           | `""`           |
| RETAIL_CATALOG_PERSISTENCE_DB_NAME         | Database name                                                   | `catalogdb`    |
| RETAIL_CATALOG_PERSISTENCE_USER            | Database user                                                   | `catalog_user` |
| RETAIL_CATALOG_PERSISTENCE_PASSWORD        | Database password                                               | `""`           |
| RETAIL_CATALOG_PERSISTENCE_CONNECT_TIMEOUT | Database connection timeout in seconds                          | `5`            |

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

**Short version:** build and run the binary. It starts with built-in sample data. **You do not need MySQL** to try it.

### 1. What you need

- Go 1.23 or newer. `go.mod` asks for toolchain `go1.24.5`; if your Go is older, it downloads the right one by itself on the first build (needs internet).

### 2. Start it

```
cd src/catalog
go build -o catalog main.go
./catalog
```

### 3. Check that it works

```
curl localhost:8080/health                  # OK
curl localhost:8080/catalog/size            # {"size":12}
curl localhost:8080/catalog/products        # list of products
curl localhost:8080/catalog/tags            # list of tags
```

> The path is `/catalog/...`. Older notes that say `/catalogue` are wrong and return 404.

### 4. Run with the real MySQL database (optional)

```
docker run -d --name catalog-mysql -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=catalogdb \
  -e MYSQL_USER=catalog_user -e MYSQL_PASSWORD=testing mysql:8

RETAIL_CATALOG_PERSISTENCE_PROVIDER=mysql \
RETAIL_CATALOG_PERSISTENCE_ENDPOINT=localhost:3306 \
RETAIL_CATALOG_PERSISTENCE_DB_NAME=catalogdb \
RETAIL_CATALOG_PERSISTENCE_USER=catalog_user \
RETAIL_CATALOG_PERSISTENCE_PASSWORD=testing \
./catalog
```

The log says `Using mysql database localhost:3306`, and the tables are filled on first start. Wait about 20 seconds after `docker run` for MySQL to be ready. Clean up with `docker rm -f catalog-mysql`.

### 5. Docker

The `Dockerfile` for this service is written in Phase 2 (issue #7). Until then, use the steps above.

### Common problems

| You see | Cause and fix |
| ------- | ------------- |
| First build is slow and prints `go: downloading ...` | Normal. Go is fetching modules and the toolchain. |
| `bind: address already in use` | Port 8080 is busy. Use `PORT=8081 ./catalog`. |
| `Error 1045` or `connection refused` with MySQL | MySQL is still starting, or the user and password do not match the ones in `docker run`. |

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
