
# Microservice Architecture with Ruby on Rails

## 🧠 What Is a Microservice Architecture?

A **microservice architecture** is a way of building software systems as a **collection of small, independent services**, each responsible for a specific business capability.

Instead of one big monolithic Rails app, you split your app into multiple **smaller Rails services**, each with its own:
- Database
- Codebase
- Deployment
- CI/CD pipeline

These services **communicate via APIs** (often REST or gRPC) or **message queues** (like Kafka, RabbitMQ, or Redis Streams).

---

## 🧩 Example — From Monolith to Microservices

Let’s say you have a big e-commerce app:

### Monolithic Rails App
```
app/
 ├─ models/
 ├─ controllers/
 ├─ views/
 └─ services/
```

All your domains — users, products, orders, payments — are in one app and one database.

---

### Microservices Version

You break it into smaller Rails apps:

| Microservice | Responsibility | Tech Stack | API Example |
|---------------|----------------|-------------|--------------|
| **Users Service** | Handles registration, login, profiles | Rails + PostgreSQL | `POST /users`, `POST /auth/login` |
| **Products Service** | Manages products, categories, stock | Rails + PostgreSQL | `GET /products`, `POST /products` |
| **Orders Service** | Handles checkout, order creation | Rails + PostgreSQL | `POST /orders`, `GET /orders/:id` |
| **Payments Service** | Processes payments, refunds | Rails + Stripe API | `POST /payments` |

Each one runs independently, for example:
```
users-service:     localhost:3001
products-service:  localhost:3002
orders-service:    localhost:3003
payments-service:  localhost:3004
```

They talk via **HTTP JSON APIs** or **message queues**.

---

## ⚙️ Example: Rails Microservice — “Users Service”

Let’s scaffold a minimal “Users Service”.

### 1. Create the service
```bash
rails new users_service --api
cd users_service
```

### 2. Generate User model & controller
```bash
rails g model User name:string email:string password_digest:string
rails g controller Api::V1::Users
rails db:migrate
```

### 3. Add JWT Authentication (optional)
Use the gem `jwt` to issue and verify tokens:
```ruby
# app/controllers/api/v1/auth_controller.rb
class Api::V1::AuthController < ApplicationController
  def login
    user = User.find_by(email: params[:email])
    if user&.authenticate(params[:password])
      token = JWT.encode({ user_id: user.id }, Rails.application.secret_key_base)
      render json: { token: token }, status: :ok
    else
      render json: { error: 'Invalid credentials' }, status: :unauthorized
    end
  end
end
```

---

## 🔗 Example: Communication Between Services

Suppose the **Orders Service** needs user info.

In the `orders_service`, you can use an HTTP client like `Faraday` or `HTTParty`:

```ruby
# app/services/user_client.rb
class UserClient
  BASE_URL = ENV.fetch('USERS_SERVICE_URL', 'http://localhost:3001')

  def self.find_user(user_id)
    response = Faraday.get("#{BASE_URL}/api/v1/users/#{user_id}")
    JSON.parse(response.body) if response.success?
  end
end
```

Then in your `Order` creation logic:
```ruby
user = UserClient.find_user(params[:user_id])
if user.nil?
  render json: { error: 'User not found' }, status: :not_found
else
  # proceed with order creation
end
```

---

## 🧱 Tools Commonly Used with Rails Microservices

| Purpose | Common Tools |
|----------|---------------|
| **API communication** | Faraday, HTTParty, gRPC, GraphQL |
| **Service discovery** | Kubernetes, Consul |
| **Asynchronous messaging** | Sidekiq, RabbitMQ, Kafka |
| **API Gateway** | NGINX, Kong, Traefik |
| **Auth & tokens** | JWT, Doorkeeper (OAuth2) |
| **Observability** | Prometheus, Grafana, Elastic APM, OpenTelemetry |

---

## 🚀 Summary

| Concept | Description |
|----------|--------------|
| **Goal** | Build independently deployable, small services |
| **Each microservice** | Has its own DB, logic, and API |
| **Rails role** | Each service can be a Rails API-only app |
| **Communication** | Via HTTP or message queues |
| **Deployment** | Containerized (Docker + Kubernetes) |

---

### ✅ Next Step

You can extend this by creating two Rails services — `users_service` and `orders_service` — and connect them via REST API or message queue using Docker Compose.
