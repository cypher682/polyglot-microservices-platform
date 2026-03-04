# Polyglot Microservices Platform

![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=flat&logo=ansible&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws&logoColor=white)

> A production-style polyglot microservices application — five services written in four different languages, containerized with Docker, deployed to AWS EC2 via a fully automated CI/CD pipeline with Terraform infrastructure provisioning, Ansible configuration management, and infrastructure drift detection.

---

## Architecture

```
                    ┌─────────────────────────────────┐
                    │       GitHub Actions CI/CD        │
                    │  ┌──────────┐  ┌──────────────┐  │
                    │  │  Infra   │  │     App      │  │
                    │  │Terraform │  │  Docker SSH  │  │
                    │  │+ Ansible │  │   Deploy     │  │
                    │  └────┬─────┘  └──────┬───────┘  │
                    └───────┼───────────────┼──────────┘
                            │               │
                            ▼               ▼
                    ┌───────────────────────────────────┐
                    │         AWS EC2 (Ubuntu)           │
                    │                                    │
                    │   ┌─────────────────────────┐     │
                    │   │      Nginx (Reverse      │     │
                    │   │         Proxy)            │     │
                    │   └─────┬──────────┬─────────┘     │
                    │         │          │               │
          ┌─────────┼─────────┘          └──────────┐   │
          │         │                               │   │
          ▼         ▼                               ▼   │
   ┌──────────┐ ┌──────────┐  ┌──────────┐  ┌──────────┐│
   │ Frontend │ │ Auth API │  │ Todos API│  │ Users API││
   │  Vue.js  │ │    Go    │  │  Node.js │  │  Java    ││
   │  :3000   │ │  :8081   │  │   :8082  │  │  Spring  ││
   └──────────┘ └──────────┘  └────┬─────┘  └──────────┘│
                                    │                    │
                              ┌─────▼─────┐              │
                              │Log Processor│             │
                              │  Python   │              │
                              │  + Redis  │              │
                              └───────────┘              │
                    └───────────────────────────────────┘
```

---

## Tech Stack

| Service               | Language / Framework    | Purpose                                |
| --------------------- | ----------------------- | -------------------------------------- |
| **Frontend**          | Vue.js                  | User interface                         |
| **Auth API**          | Go                      | JWT token issuance & validation        |
| **Todos API**         | Node.js                 | CRUD operations on todo items          |
| **Users API**         | Java (Spring Boot)      | User profile management                |
| **Log Processor**     | Python                  | Reads from Redis queue, logs to stdout |
| **Message Broker**    | Redis                   | Async log event queue                  |
| **Reverse Proxy**     | Nginx                   | TLS termination, routing               |
| **Infrastructure**    | Terraform + Ansible     | EC2 provisioning + configuration       |
| **CI/CD**             | GitHub Actions          | Two-pipeline automation                |
| **Container Runtime** | Docker + Docker Compose | Multi-service orchestration            |

---

## CI/CD Pipelines

This project uses two separate GitHub Actions pipelines for separation of concerns:

### 1. Infrastructure Pipeline (`infrastructure.yml`)

Triggered on changes to `infra/terraform/**` or `infra/ansible/**`.

```
Push → Terraform Refresh → Terraform Plan → Drift Detection
                                │
                    ┌───────────┴───────────┐
                    │ Drift Detected?        │
                    │ YES → Email Alert +    │
                    │        Terraform Apply │
                    │ NO  → Terraform Apply  │
                    └───────────────────────┘
                                │
                    Ansible Playbook → EC2 Setup
```

**Key features:**

- **Drift detection** — automatically detects infrastructure configuration drift and emails an alert with the Terraform plan attached
- **Conditional apply** — applies changes only after drift alert is sent and acknowledged
- **Ansible post-provisioning** — configures the EC2 host after Terraform creates it

### 2. Application Pipeline (`application.yml`)

Triggered on changes to any service directory or `docker-compose.yml`.

```
Push → SSH into EC2 → git pull → docker compose down
     → docker compose up --build → Health check → Email notification
```

---

## Getting Started

### Prerequisites

- AWS account with CLI configured
- Terraform Cloud account (`cypher682-org` organization)
- Docker & Docker Compose
- GitHub repository with the following secrets configured:

| Secret                  | Description                            |
| ----------------------- | -------------------------------------- |
| `AWS_ACCESS_KEY_ID`     | AWS credentials                        |
| `AWS_SECRET_ACCESS_KEY` | AWS credentials                        |
| `TF_CLOUD_TOKEN`        | Terraform Cloud API token              |
| `SSH_PRIVATE_KEY`       | EC2 SSH private key                    |
| `GMAIL_USER`            | Gmail address for deploy notifications |
| `GMAIL_APP_PASSWORD`    | Gmail app password                     |
| `ALERT_EMAIL`           | Recipient for drift/deploy alerts      |

### Run Locally

```bash
git clone https://github.com/cypher682/polyglot-microservices-platform.git
cd polyglot-microservices-platform

# Copy and fill in environment variables
cp .env.example .env

# Start all services
docker compose up --build
```

**Service URLs (local):**
| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| Auth API | http://localhost:8081 |
| Todos API | http://localhost:8082 |
| Users API | http://localhost:8083 |

> **Note:** Three sets of login credentials are pre-configured in `.env` for testing.

### Deploy to AWS (via CI/CD)

Push to `main` — the pipelines handle everything.

To trigger manually:

```bash
# Trigger infrastructure pipeline
gh workflow run infrastructure.yml

# Trigger app pipeline
gh workflow run application.yml
```

---

## Project Structure

```
polyglot-microservices-platform/
├── .github/
│   └── workflows/
│       ├── infrastructure.yml    ← Terraform + Ansible pipeline
│       └── application.yml       ← Docker Compose deploy pipeline
├── frontend/                     ← Vue.js SPA
├── auth-api/                     ← Go JWT service
├── todos-api/                    ← Node.js CRUD service
├── users-api/                    ← Java Spring Boot service
├── log-message-processor/        ← Python Redis consumer
├── infra/
│   ├── terraform/                ← AWS EC2 + VPC IaC
│   └── ansible/                  ← EC2 post-provisioning playbook
├── docker-compose.yml            ← Multi-service orchestration
└── .env                          ← Environment variables
```

---

## Skills Demonstrated

- **Polyglot containerization** — Dockerizing services across Go, Node.js, Java, Python, and Vue.js
- **Infrastructure as Code** — Terraform for AWS EC2 provisioning with remote state (Terraform Cloud)
- **Configuration management** — Ansible for automated server setup post-provisioning
- **Multi-pipeline CI/CD** — Separate GitHub Actions workflows for infra vs. application changes
- **Infrastructure drift detection** — Automated detection and email alerting on config drift
- **Async messaging** — Redis queue for decoupled log processing between services
- **Reverse proxy configuration** — Nginx routing across multiple containerized services

---

## Author

**Suleiman Abdulrahman** — DevOps & Cloud Engineer  
[github.com/cypher682](https://github.com/cypher682) | [linkedin.com/in/suleiman-abdulrahman-dev](https://linkedin.com/in/suleiman-abdulrahman-dev)

---

_Completed during HNG13 DevOps Internship (2025)_
