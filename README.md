# Microservices Lab

Laboratório de estudo de comunicação entre microsserviços utilizando Node.js, Java/Spring Boot, RabbitMQ, PostgreSQL e MongoDB — tudo orquestrado com Docker Compose.

---

## Diagrama do Sistema

<img src="./files/diagram.jpeg">

---

## Serviços

| Serviço | Tecnologia | Banco de Dados | Porta |
|---------|-----------|---------------|-------|
| **auth-api** | Node.js 14 + Express | PostgreSQL 11 | 8080 |
| **product-api** | Java 17 + Spring Boot 3 | PostgreSQL 11 | 8081 |
| **sales-api** | Node.js 14 + Express | MongoDB | 8082 |

### auth-api

Serviço responsável por autenticação e gerenciamento de usuários.

- **Stack:** Node.js, Express, Sequelize, JWT, bcrypt, PostgreSQL
- **Funcionalidades:** Criação e autenticação de usuários, geração de tokens JWT

### product-api

Serviço responsável pelo catálogo de produtos, categorias e fornecedores.

- **Stack:** Java 17, Spring Boot 3.1.1, Spring Data JPA, RabbitMQ (AMQP), PostgreSQL, Gradle
- **Funcionalidades:** CRUD de produtos, categorias e fornecedores; atualização de estoque via mensageria
- **Dados iniciais:** Categorias (HQs, Filmes, Livros), Fornecedores (Panini Comics, Amazon) e 3 produtos de exemplo

### sales-api

Serviço responsável pelo gerenciamento de pedidos (orders).

- **Stack:** Node.js, Express, Mongoose, RabbitMQ (amqplib), Axios, JWT, MongoDB
- **Funcionalidades:** Criação e consulta de pedidos, comunicação assíncrona com product-api para atualização de estoque

---

## Comunicação entre Serviços

```
auth-api  ──(HTTP)──►  product-api
                           │
                    ┌──────┴──────┐
                 (HTTP)       (RabbitMQ)
                    │              │
                    ▼              ▼
                sales-api ◄──► product-api
```

- **HTTP:** sales-api consulta product-api para buscar informações de produtos
- **RabbitMQ (mensageria assíncrona):**
  - Exchange: `product.topic`
  - Fila de atualização de estoque: `product-stock-update.queue`
  - Fila de confirmação de venda: `sales-confirmation.queue`

---

## Infraestrutura (Docker Compose)

| Container | Imagem | Porta(s) |
|-----------|--------|----------|
| `postgres-auth` | postgres:11 | 5432 |
| `postgres-products` | postgres:11 | 5433 |
| `mongo-sales` | mongo:latest | 27017 |
| `rabbitMQ` | rabbitmq:3-management | 5672, 15672 |
| `auth-api` | build local | 8080 |
| `product-api` | build local | 8081 |
| `sales-api` | build local | 8082 |

**RabbitMQ Management UI:** http://localhost:15672 (guest/guest)

---

## Pré-requisitos

- [Docker](https://docs.docker.com/get-docker/) e [Docker Compose](https://docs.docker.com/compose/install/)

> Para desenvolvimento local (sem Docker):
> - [Node.js](https://nodejs.org/en/) (v14+)
> - [Yarn](https://classic.yarnpkg.com/en/docs/install)
> - [Java 17](https://jdk.java.net/17/) e [Gradle](https://gradle.org/install/)
> - [Git](https://git-scm.com/downloads)
> - [VSCode](https://code.visualstudio.com/download) ou [IntelliJ IDEA](https://www.jetbrains.com/pt-br/idea/)
> - [DBeaver](https://dbeaver.io/download/) (cliente SQL)
> - [MongoDB Shell](https://www.mongodb.com/try/download/shell)

---

## Como Executar

### Com Docker (recomendado)

```bash
# Na raiz do projeto
docker-compose up
```

Todos os serviços serão iniciados automaticamente na ordem correta, com os bancos de dados e o RabbitMQ devidamente configurados.

### Desenvolvimento local

Cada serviço possui seus próprios scripts de desenvolvimento:

```bash
# auth-api e sales-api
yarn startDev

# product-api
./gradlew bootRun
```

---

## Estrutura do Projeto

```
microservices-lab/
├── auth-api/          # Serviço de autenticação (Node.js)
├── product-api/       # Serviço de produtos (Java/Spring Boot)
├── sales-api/         # Serviço de pedidos (Node.js)
├── mongodb/           # Script de inicialização do MongoDB
├── files/             # Arquivos de documentação e diagramas
└── docker-compose.yaml
```