# Internal AI Chat Platform

An enterprise-ready internal AI chat platform built around **Open WebUI**.

The goal of this project is to provide employees with a secure, centralized AI assistant that can connect to cloud-based AI models, local models, and internal company knowledge.

---

## 📋 Table of Contents

* [Overview](#-overview)
* [Architecture](#-architecture)
* [Features](#-features)
* [Technology Stack](#-technology-stack)
* [Requirements](#-requirements)
* [Installation](#-installation)
* [Docker Installation](#-docker-installation)
* [Docker Compose](#-docker-compose)
* [Configuration](#-configuration)
* [OpenAI Integration](#-openai-integration)
* [Ollama Integration](#-ollama-integration)
* [Internal Knowledge & RAG](#-internal-knowledge--rag)
* [Authentication](#-authentication)
* [Security](#-security)
* [HTTPS & Reverse Proxy](#-https--reverse-proxy)
* [Backups](#-backups)
* [Updates](#-updates)
* [Monitoring](#-monitoring)
* [Production Architecture](#-production-architecture)
* [Recommended Roadmap](#-recommended-roadmap)
* [Troubleshooting](#-troubleshooting)
* [Project Structure](#-project-structure)
* [License](#-license)

---

# 🚀 Overview

This project provides a private AI chat environment for internal company use.

Employees can access a ChatGPT-like interface from their browser and interact with AI models approved by the organization.

The platform can be configured to use:

* Cloud AI providers such as OpenAI
* Local AI models through Ollama
* Internal company documents
* Retrieval-Augmented Generation (RAG)
* Enterprise authentication
* Internal APIs and business applications

The platform is designed to be deployed on the company's own infrastructure.

---

# 🏗️ Architecture

The recommended architecture is a hybrid setup combining Open WebUI, cloud AI models, local models, and internal company knowledge.

```text
                         ┌───────────────────┐
                         │     Employees     │
                         └─────────┬─────────┘
                                   │
                                   │ HTTPS
                                   ▼
                         ┌───────────────────┐
                         │ Authentication    │
                         │ SSO / LDAP / OIDC │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │  Reverse Proxy    │
                         │ Nginx / Traefik   │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │    Open WebUI     │
                         │                   │
                         │ Chat              │
                         │ Users             │
                         │ Documents         │
                         │ RAG               │
                         └───────┬───┬───────┘
                                 │   │
                    ┌────────────┘   └────────────┐
                    ▼                             ▼
           ┌─────────────────┐           ┌─────────────────┐
           │    OpenAI API   │           │     Ollama      │
           │      Cloud      │           │      Local      │
           └─────────────────┘           └────────┬────────┘
                                                   │
                                                   ▼
                                          ┌─────────────────┐
                                          │ Local AI Models │
                                          └─────────────────┘

                         Internal Knowledge
                                │
                                ▼
                         ┌─────────────────┐
                         │       RAG       │
                         │ Vector Database │
                         └─────────────────┘
                                │
                                ▼
                         Company Documents
```

---

# ✨ Features

## AI Chat

* Chat with approved AI models
* Multiple models/providers
* Conversation history
* Streaming responses
* Model selection

## Internal Knowledge

The platform can be connected to internal documentation such as:

* PDF files
* Word documents
* Excel files
* Procedures
* Product documentation
* Internal FAQs
* Technical documentation
* HR documentation

## RAG

Retrieval-Augmented Generation allows the AI to retrieve relevant information from internal company documents before generating an answer.

## Enterprise Authentication

The platform can be integrated with enterprise identity systems such as:

* Microsoft Entra ID
* Active Directory
* LDAP
* OIDC
* SSO

## Local AI

Sensitive workloads can be processed locally using Ollama and compatible open-source models.

## Hybrid AI

Different models can be used depending on the task:

```text
Sensitive data
      ↓
Local model

Complex reasoning
      ↓
Cloud model

Internal documentation
      ↓
RAG
```

---

# 🧰 Technology Stack

| Component               | Technology                                      |
| ----------------------- | ----------------------------------------------- |
| Web UI                  | Open WebUI                                      |
| Containerization        | Docker                                          |
| Container orchestration | Docker Compose                                  |
| Cloud AI                | OpenAI API                                      |
| Local AI                | Ollama                                          |
| Database                | Open WebUI / PostgreSQL depending on deployment |
| RAG                     | Open WebUI knowledge/RAG system                 |
| Reverse Proxy           | Nginx / Traefik / Caddy                         |
| Authentication          | OIDC / LDAP / SSO                               |
| Operating System        | Ubuntu Server recommended                       |

---

# 💻 Requirements

## Minimum Development Environment

```text
CPU:       4 cores
RAM:       8 GB
Storage:   100 GB SSD
OS:        Ubuntu 22.04+ / Ubuntu 24.04+
Docker:    Latest stable version
```

## Recommended Production Environment

```text
CPU:       8+ cores
RAM:       32 GB+
Storage:   500 GB+ SSD
Network:   1 Gbps
GPU:       Optional
OS:        Ubuntu Server 24.04 LTS
```

GPU requirements depend on the local AI models being used.

---

# 🐳 Installation

## 1. Install Docker

On Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
```

Install Docker:

```bash
sudo apt install docker.io docker-compose-plugin -y
```

Verify:

```bash
docker --version
docker compose version
```

Test Docker:

```bash
sudo docker run hello-world
```

---

# 🐳 Docker Installation

Create a dedicated directory:

```bash
sudo mkdir -p /opt/open-webui
cd /opt/open-webui
```

Run Open WebUI:

```bash
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  -e WEBUI_SECRET_KEY="$(openssl rand -hex 32)" \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Check the container:

```bash
docker ps
```

View logs:

```bash
docker logs -f open-webui
```

Open the application:

```text
http://SERVER_IP:3000
```

---

# 🐙 Docker Compose

For production environments, Docker Compose is recommended.

Create:

```text
docker-compose.yml
```

Example:

```yaml
services:

  open-webui:
    image: ghcr.io/open-webui/open-webui:main

    container_name: open-webui

    restart: unless-stopped

    ports:
      - "3000:8080"

    volumes:
      - open-webui:/app/backend/data

    environment:
      WEBUI_SECRET_KEY: "CHANGE_THIS_TO_A_LONG_RANDOM_SECRET"

volumes:
  open-webui:
```

Start the service:

```bash
docker compose up -d
```

Check status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

Stop the service:

```bash
docker compose down
```

---

# ⚙️ Configuration

After installation, access the administration interface.

Typical administration areas include:

```text
Settings
 ├── General
 ├── Users
 ├── Models
 ├── Connections
 ├── Documents
 ├── Authentication
 └── Administration
```

The exact configuration options may vary depending on the Open WebUI version.

---

# 🤖 OpenAI Integration

Open WebUI can connect to OpenAI through its API.

The standard API endpoint is:

```text
https://api.openai.com/v1
```

Configure the provider from the Open WebUI administration interface.

Architecture:

```text
Employee
   │
   ▼
Open WebUI
   │
   ▼
OpenAI API
   │
   ▼
AI Model
```

The API key should never be hard-coded into source code or committed to Git.

Use a secure secret-management mechanism where possible.

---

# 🧠 Ollama Integration

Ollama can be used to run AI models locally.

Architecture:

```text
Open WebUI
     │
     ▼
   Ollama
     │
     ▼
Local AI Model
```

Example deployment:

```text
Server 1
──────────────
Open WebUI
192.168.10.10

        │
        │ Internal network
        ▼

Server 2
──────────────
Ollama
192.168.10.20
```

Open WebUI can then connect to the Ollama server.

The Ollama endpoint is typically:

```text
http://192.168.10.20:11434
```

Do not expose the Ollama API directly to the public Internet.

---

# 📚 Internal Knowledge & RAG

RAG allows employees to ask questions about internal company information.

Example document structure:

```text
company-documents/
├── HR/
│   ├── procedures/
│   └── policies/
│
├── IT/
│   ├── documentation/
│   └── troubleshooting/
│
├── Finance/
│   └── procedures/
│
├── Sales/
│   └── products/
│
└── Management/
    └── reports/
```

The documents are processed and indexed so that relevant information can be retrieved when a user asks a question.

Example:

```text
User:
"What is the procedure for requesting vacation?"

       ↓

RAG Search

       ↓

Relevant company document

       ↓

AI Model

       ↓

Answer
```

---

# 🔐 Document Permissions

Document permissions are critical in an enterprise environment.

Sensitive documents should not be available to every user.

Example:

```text
HR
 ├── HR Group
 └── Management Group

Finance
 └── Finance Group

Sales
 └── Sales Group

IT
 └── IT Group
```

A sales employee should not automatically be able to search HR or payroll documents.

Access-control policies must therefore be designed before importing sensitive company data.

---

# 👥 Authentication

For production use, authentication should be enabled.

Recommended architecture:

```text
Employee
    │
    ▼
Corporate Identity Provider
    │
    ▼
SSO / OIDC
    │
    ▼
Open WebUI
```

Possible identity providers include:

* Microsoft Entra ID
* Active Directory
* LDAP
* OIDC-compatible identity providers

The preferred solution depends on the company's existing identity infrastructure.

---

# 🔒 Security

Security should be treated as a core requirement.

## Network Security

Recommended:

```text
Internet
   X
   │
Firewall
   │
   ▼
Internal Network
   │
   ▼
Reverse Proxy
   │
   ▼
Open WebUI
```

Do not expose these services directly to the public Internet:

```text
Ollama
PostgreSQL
Internal APIs
Vector databases
```

---

# 🔑 Secrets

Never commit secrets to Git.

Do not store:

```text
OPENAI_API_KEY=...
```

inside a public repository.

Use:

* Environment variables
* Docker secrets
* Vault
* Cloud secret managers
* Enterprise secret-management systems

---

# 🌐 HTTPS & Reverse Proxy

For production, Open WebUI should be served over HTTPS.

Example:

```text
https://ai.company.com
```

Architecture:

```text
Employee
   │
   │ HTTPS
   ▼
Nginx / Traefik
   │
   │ HTTP
   ▼
Open WebUI :3000
```

The reverse proxy should handle:

* TLS certificates
* HTTPS redirection
* Security headers
* Access control
* Optional rate limiting

---

# 💾 Backups

The Open WebUI data directory must be backed up.

Docker volume:

```text
open-webui
```

Example backup:

```bash
docker run --rm \
  -v open-webui:/data \
  -v /backup:/backup \
  ubuntu \
  tar czf /backup/open-webui-$(date +%Y-%m-%d).tar.gz /data
```

Recommended backup strategy:

```text
Daily backups
     │
     ├── 7 daily backups
     │
     └── Weekly backups
             │
             └── 4 weekly backups
```

A secondary backup location is strongly recommended.

---

# 🔄 Updates

Before updating, create a backup.

Pull the latest image:

```bash
docker compose pull
```

Restart:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Check logs:

```bash
docker compose logs -f
```

Production environments should ideally use a staging environment before updating production.

---

# 📊 Monitoring

Monitor at least:

```text
CPU
RAM
Disk
Network
Docker containers
Open WebUI
Ollama
GPU
Database
```

Recommended alerts:

```text
Disk usage > 80%
RAM usage > 90%
Container stopped
API errors
Database unavailable
GPU temperature too high
```

---

# 🏢 Production Architecture

Recommended production architecture:

```text
                         ┌─────────────────┐
                         │    Employees    │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ SSO / Identity  │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ Reverse Proxy   │
                         │ HTTPS           │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │    Open WebUI   │
                         └───────┬─┬───────┘
                                 │ │
                   ┌─────────────┘ └─────────────┐
                   ▼                             ▼
          ┌─────────────────┐           ┌─────────────────┐
          │    OpenAI API   │           │     Ollama      │
          │      Cloud      │           │      Local      │
          └─────────────────┘           └────────┬────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │ Local AI Models │
                                         └─────────────────┘

                    Internal Documents
                            │
                            ▼
                    ┌─────────────────┐
                    │      RAG        │
                    └─────────────────┘
```

---

# 🛡️ Recommended Firewall Rules

Example:

| Source     | Destination |    Port | Action |
| ---------- | ----------- | ------: | ------ |
| Users      | Open WebUI  |     443 | Allow  |
| Open WebUI | Ollama      |   11434 | Allow  |
| Open WebUI | Database    | DB port | Allow  |
| Users      | Database    | DB port | Deny   |
| Internet   | Ollama      |   11434 | Deny   |
| Internet   | Database    | DB port | Deny   |

These rules are examples and must be adapted to the company's network architecture.

---

# 🧪 Development / Staging / Production

It is recommended to maintain separate environments.

```text
Development
     │
     ▼
Staging
     │
     ▼
Production
```

Example:

```text
ai-dev.company.local
ai-staging.company.local
ai.company.com
```

Never test experimental configurations directly in production.

---

# 🗺️ Recommended Roadmap

## Phase 1 — Proof of Concept

```text
Docker
  ↓
Open WebUI
  ↓
OpenAI
```

Goal: validate the AI chat experience.

---

## Phase 2 — Authentication

```text
Open WebUI
  +
SSO
  +
Users
  +
Groups
```

---

## Phase 3 — Internal Knowledge

```text
PDF
Word
Excel
Procedures
     ↓
RAG
     ↓
AI
```

---

## Phase 4 — Security

```text
HTTPS
Firewall
SSO
Backups
Monitoring
```

---

## Phase 5 — Local AI

```text
Open WebUI
     ↓
Ollama
     ↓
Local Models
```

---

## Phase 6 — Business Integrations

The AI platform can later be connected to internal applications through APIs.

Potential integrations:

```text
ERP
CRM
HR
Ticketing
SharePoint
Databases
Internal APIs
```

Example:

```text
User
  │
  ▼
AI Assistant
  │
  ▼
Internal API
  │
  ▼
ERP / CRM
  │
  ▼
Result
  │
  ▼
AI Response
```

---

# 🧩 Example Use Cases

## HR

> "What is the procedure for requesting vacation?"

## IT

> "How do I configure the corporate VPN?"

## Sales

> "Summarize the documentation for Product X."

## Management

> "Summarize this month's reports."

## Support

> "Find the procedure corresponding to this error."

---

# ⚠️ AI Usage Policy

The internal AI assistant should not be considered an autonomous decision-making system.

Users should verify AI-generated information before using it for important decisions.

The system should not be used to:

* Share passwords
* Share authentication tokens
* Store secrets
* Make unreviewed HR decisions
* Make final legal decisions
* Make final financial decisions
* Process sensitive information without authorization

The company should define its own AI governance and data-handling policy.

---

# 🔧 Troubleshooting

## Open WebUI is not accessible

Check the container:

```bash
docker ps
```

Check logs:

```bash
docker logs open-webui
```

Check port:

```bash
sudo ss -tulpn | grep 3000
```

---

## Container keeps restarting

Check:

```bash
docker logs open-webui
```

Check disk space:

```bash
df -h
```

Check Docker:

```bash
systemctl status docker
```

---

## Ollama connection fails

Check that Ollama is running:

```bash
curl http://OLLAMA_SERVER:11434
```

Check network connectivity:

```bash
ping OLLAMA_SERVER
```

Check firewall rules.

---

# 📁 Project Structure

A possible repository structure:

```text
internal-ai-chat/
│
├── README.md
│
├── docker/
│   ├── docker-compose.yml
│   └── .env.example
│
├── nginx/
│   └── nginx.conf
│
├── scripts/
│   ├── backup.sh
│   ├── restore.sh
│   └── update.sh
│
├── docs/
│   ├── architecture.md
│   ├── security.md
│   ├── deployment.md
│   └── operations.md
│
└── backups/
```

---

# 📌 Production Checklist

## Infrastructure

* [ ] Server configured
* [ ] Docker installed
* [ ] Storage configured
* [ ] Backup configured
* [ ] Monitoring configured

## Open WebUI

* [ ] Open WebUI deployed
* [ ] Persistent volume configured
* [ ] Secret key configured
* [ ] Administrator account created
* [ ] AI provider configured

## Authentication

* [ ] SSO configured
* [ ] User groups configured
* [ ] Administrator access restricted

## Security

* [ ] HTTPS enabled
* [ ] Firewall configured
* [ ] Ollama not publicly exposed
* [ ] Database not publicly exposed
* [ ] Secrets protected
* [ ] Backups encrypted if required

## AI

* [ ] Primary model configured
* [ ] Secondary model configured
* [ ] RAG configured
* [ ] Documents indexed
* [ ] Document permissions reviewed

## Operations

* [ ] Backup procedure documented
* [ ] Restore procedure tested
* [ ] Update procedure documented
* [ ] Monitoring enabled
* [ ] Incident procedure documented

---

# 🎯 Final Recommended Architecture

For a serious internal deployment, the recommended target architecture is:

```text
                    ┌─────────────────────┐
                    │     Employees       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Corporate SSO       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Reverse Proxy       │
                    │ HTTPS               │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     Open WebUI      │
                    │                     │
                    │ Chat                │
                    │ Users               │
                    │ Models              │
                    │ Knowledge           │
                    │ RAG                 │
                    └───────┬─────┬───────┘
                            │     │
                 ┌──────────┘     └──────────┐
                 ▼                           ▼
        ┌─────────────────┐         ┌─────────────────┐
        │   OpenAI API    │         │     Ollama      │
        │      Cloud      │         │      Local      │
        └─────────────────┘         └────────┬────────┘
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │ Local AI Models │
                                    └─────────────────┘

                         ┌─────────────────────┐
                         │ Internal Documents  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │       RAG           │
                         │ Vector Search       │
                         └─────────────────────┘
```

---

# 📚 Documentation

Official Open WebUI documentation:

* [Open WebUI Documentation](https://docs.openwebui.com/)
* [Quick Start](https://docs.openwebui.com/getting-started/quick-start/)
* [Providers](https://docs.openwebui.com/getting-started/quick-start/connect-a-provider/)
* [OpenAI Integration](https://docs.openwebui.com/getting-started/quick-start/connect-a-provider/starting-with-openai/)
* [Ollama Integration](https://docs.openwebui.com/getting-started/quick-start/connect-a-provider/starting-with-ollama/)

---

# 📄 License

This repository contains the deployment and configuration documentation for the internal AI platform.

Open WebUI and the other components used by the deployment are subject to their respective licenses.

Before deploying in a commercial environment, review the license terms of every component and AI model being used.

---

# 👥 Project Ownership

**Project:** Internal AI Chat Platform

**Purpose:** Secure enterprise AI assistant

**Primary Interface:** Open WebUI

**Deployment:** Docker / Docker Compose

**AI Providers:** OpenAI / Ollama

**Environment:** Internal Enterprise Infrastructure

---

## Status

🚧 **Project Status: In Development**

The production architecture should be validated by the organization's IT, security, and compliance teams before deployment.
