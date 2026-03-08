# Microservices Lab

A study lab for microservices communication using Node.js, Java/Spring Boot, RabbitMQ, PostgreSQL, and MongoDB — all orchestrated with Docker Compose.

---

## System Diagram

<img src="./files/diagram.jpeg">

---

## Services Overview

| Service | Technology | Database | Port |
|---------|-----------|----------|------|
| **auth-api** | Node.js 14 + Express | PostgreSQL 11 | 8080 |
| **product-api** | Java 17 + Spring Boot 3 | PostgreSQL 11 | 8081 |
| **sales-api** | Node.js 14 + Express | MongoDB | 8082 |

---

### auth-api

Responsible for user authentication and management.

- **Stack:** Node.js, Express, Sequelize, JWT, bcrypt, PostgreSQL
- **Features:** User registration and authentication, JWT token generation

#### Endpoints

| Method | Path | Description | Auth required |
|--------|------|-------------|---------------|
| `POST` | `/api/user/auth` | Authenticate user and return JWT token | No |
| `GET` | `/api/user/email/:email` | Find user by email address | Yes |

---

### product-api

Responsible for the product catalog, categories, and suppliers.

- **Stack:** Java 17, Spring Boot 3.1.1, Spring Data JPA, RabbitMQ (AMQP), PostgreSQL, Gradle
- **Features:** Full CRUD for products, categories, and suppliers; asynchronous stock updates via messaging
- **Seed data:** Categories (Comics, Movies, Books), Suppliers (Panini Comics, Amazon), and 3 sample products

#### Endpoints

**Products** — base path `/api/product`

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/product` | Create a new product |
| `GET` | `/api/product` | List all products |
| `GET` | `/api/product/:id` | Find product by ID |
| `GET` | `/api/product/name/:name` | Search products by name |
| `GET` | `/api/product/category/:categoryId` | List products by category |
| `GET` | `/api/product/supplier/:supplierId` | List products by supplier |
| `PUT` | `/api/product/:id` | Update product |
| `DELETE` | `/api/product/:id` | Delete product |
| `POST` | `/api/product/check-stock` | Check available stock for a list of products |
| `GET` | `/api/product/:id/sales` | Get sales associated with a product |

**Categories** — base path `/api/category`

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/category` | Create a new category |
| `GET` | `/api/category` | List all categories |
| `GET` | `/api/category/:id` | Find category by ID |
| `GET` | `/api/category/description/:description` | Search categories by description |
| `PUT` | `/api/category/:id` | Update category |
| `DELETE` | `/api/category/:id` | Delete category |

**Suppliers** — base path `/api/supplier`

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/supplier` | Create a new supplier |
| `GET` | `/api/supplier` | List all suppliers |
| `GET` | `/api/supplier/:id` | Find supplier by ID |
| `GET` | `/api/supplier/name/:name` | Search suppliers by name |
| `PUT` | `/api/supplier/:id` | Update supplier |
| `DELETE` | `/api/supplier/:id` | Delete supplier |

---

### sales-api

Responsible for order management.

- **Stack:** Node.js, Express, Mongoose, RabbitMQ (amqplib), Axios, JWT, MongoDB
- **Features:** Order creation and retrieval, asynchronous communication with product-api for stock updates

#### Endpoints

| Method | Path | Description | Auth required |
|--------|------|-------------|---------------|
| `POST` | `/api/order/create` | Create a new order | Yes |
| `GET` | `/api/order/:id` | Find order by ID | Yes |
| `GET` | `/api/orders` | List all orders | Yes |
| `GET` | `/api/orders/product/:productId` | List orders containing a specific product | Yes |

---

## Service Communication

```
auth-api  ──(HTTP)──►  product-api
                           │
                    ┌──────┴──────┐
                 (HTTP)       (RabbitMQ)
                    │              │
                    ▼              ▼
                sales-api ◄──► product-api
```

- **HTTP:** sales-api queries product-api to fetch product information
- **RabbitMQ (asynchronous messaging):**
  - Exchange: `product.topic`
  - Stock update queue: `product-stock-update.queue`
  - Sales confirmation queue: `sales-confirmation.queue`

---

## Infrastructure (Docker Compose)

| Container | Image | Port(s) |
|-----------|-------|---------|
| `postgres-auth` | postgres:11 | 5432 |
| `postgres-products` | postgres:11 | 5433 |
| `mongo-sales` | mongo:latest | 27017 |
| `rabbitMQ` | rabbitmq:3-management | 5672, 15672 |
| `auth-api` | local build | 8080 |
| `product-api` | local build | 8081 |
| `sales-api` | local build | 8082 |

**RabbitMQ Management UI:** http://localhost:15672 (guest/guest)

---

## Environment Variables

Each service is configured through environment variables set in `docker-compose.yaml`.

### auth-api

| Variable | Description |
|----------|-------------|
| `PORT` | Server port (default: `8080`) |
| `API_SECRET` | JWT secret (base64-encoded) |
| `DB_HOST` | PostgreSQL host |
| `DB_NAME` | Database name (`auth-db`) |
| `DB_USER` | Database user |
| `DB_PASSWORD` | Database password |
| `DB_PORT` | Database port |
| `NODE_ENV` | Environment (`container` or `development`) |

### product-api

| Variable | Description |
|----------|-------------|
| `PORT` | Server port (default: `8081`) |
| `API_SECRET` | JWT secret (base64-encoded) |
| `DB_HOST` | PostgreSQL host |
| `DB_NAME` | Database name (`product-db`) |
| `DB_USER` | Database user |
| `DB_PASSWORD` | Database password |
| `DB_PORT` | Database port |
| `RABBIT_MQ_HOST` | RabbitMQ host |
| `RABBIT_MQ_PORT` | RabbitMQ port |
| `RABBIT_MQ_USER` | RabbitMQ user |
| `RABBIT_MQ_PASSWORD` | RabbitMQ password |
| `SALES_HOST` | sales-api hostname |
| `SALES_PORT` | sales-api port |

### sales-api

| Variable | Description |
|----------|-------------|
| `PORT` | Server port (default: `8082`) |
| `API_SECRET` | JWT secret (base64-encoded) |
| `MONGO_DB_URL` | MongoDB connection URL |
| `RABBIT_MQ_URL` | RabbitMQ connection URL |
| `PRODUCT_API_URL` | product-api base URL |
| `NODE_ENV` | Environment (`container` or `development`) |

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)

> For local development (without Docker):
> - [Node.js](https://nodejs.org/en/) (v14+)
> - [Yarn](https://classic.yarnpkg.com/en/docs/install)
> - [Java 17](https://jdk.java.net/17/) and [Gradle](https://gradle.org/install/)
> - [Git](https://git-scm.com/downloads)
> - [VSCode](https://code.visualstudio.com/download) or [IntelliJ IDEA](https://www.jetbrains.com/pt-br/idea/)
> - [DBeaver](https://dbeaver.io/download/) (SQL client)
> - [MongoDB Shell](https://www.mongodb.com/try/download/shell)

---

## Getting Started

### With Docker (recommended)

```bash
# From the project root
docker-compose up
```

All services will start automatically in the correct order, with databases and RabbitMQ properly configured.

### Local development

Each service has its own development scripts:

```bash
# auth-api and sales-api
yarn startDev

# product-api
./gradlew bootRun
```

---

## Project Structure

```
microservices-lab/
├── auth-api/          # Authentication service (Node.js)
├── product-api/       # Product catalog service (Java/Spring Boot)
├── sales-api/         # Order management service (Node.js)
├── mongodb/           # MongoDB initialization script
├── files/             # Documentation files and diagrams
└── docker-compose.yaml
```