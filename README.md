# Cyberwatch Local Lab

*OpenCTI + Ollama + OpenClaw*

A **reproducible local Cyber Threat Intelligence (CTI) lab** combining **OpenCTI 6.5**, **Elasticsearch 8**, **Ollama**, and an **OpenClaw gateway**, designed for experimenting with **AI-augmented threat intelligence** on a single Linux host.

> ⚠️ This repository is intended for **local lab and research purposes only**.  
> It is **not** a hardened production deployment.

---

## ✨ Features

This lab provides a self-hosted CTI stack composed of:

- **OpenCTI 6.5**
  - Platform
  - Worker
- **Elasticsearch 8.14**
  - Single-node
  - HTTP only (no TLS)
- **MinIO**
  - S3-compatible object storage
- **RabbitMQ**
  - Message broker
- **Redis**
  - Cache and internal queues
- **Ollama**
  - Local LLM server (HTTP)

🔹 The **OpenClaw gateway is intentionally not containerized**.  
It is expected to run on the host (or another machine) and connect to Ollama over HTTP.

---

## 🧱 Architecture

- OpenCTI uses:
  - Elasticsearch for indexing
  - MinIO for file storage
  - RabbitMQ and Redis for messaging and caching
- Ollama exposes a local LLM API on port **11434**
- OpenClaw (running outside Docker):
  - Uses Ollama as its LLM backend
  - Can later integrate with OpenCTI via API

Typical deployment:  
A **single Linux VM** (Ubuntu Server or similar) running Docker and Docker Compose.

---

## 🖥️ Requirements

- Linux host (tested on **Ubuntu Server**)
- Docker
- Docker Compose

### Recommended minimum resources

- **4 vCPU**
- **8 GB RAM**
- **30 GB free disk space**

---

## ⚙️ Configuration

All sensitive values in `docker-compose.yml` are placeholders and **must be changed**:

- `CHANGE_ME_ADMIN_PASSWORD`
- `CHANGE_ME_MINIO_PASSWORD`
- `CHANGE_ME_RABBITMQ_PASSWORD`
- `CHANGE_ME_VT_KEY`

### Environment variables

You can manage secrets centrally using a `.env` file:

cp .env.example .env
# Edit .env and replace all CHANGE_ME values
👤 Default admin (lab only)
The default configuration uses:

Email: changeme@example.com

Password: CHANGE_ME_ADMIN_PASSWORD

API Token: 11111111-1111-1111-1111-111111111111

⚠️ These credentials are strictly for local lab usage
and must be replaced in any real or exposed environment.

🚀 Quick start
Clone the repository
bash
Copier le code
git clone https://github.com/Azimen59/cyberwatch-local-lab.git
cd cyberwatch-local-lab
(Optional) Configure environment variables
bash
Copier le code
cp .env.example .env
# Edit .env with your own secrets
Start the stack
bash
Copier le code
docker-compose up -d
docker-compose ps
🌐 Accessing services
OpenCTI
URL: http://<VM_IP>:8080

Admin email: changeme@example.com

Admin password: CHANGE_ME_ADMIN_PASSWORD

➡️ Change these values before exposing anything outside your lab.

Ollama (local LLM)
API endpoint: http://<VM_IP>:11434

Basic test:

bash
Copier le code
curl http://localhost:11434/api/tags
You can pull models (e.g. llama3, mistral) using the Ollama CLI.

🧠 OpenClaw gateway (outside Docker)
Once OpenClaw is installed on the host (or another machine on the same network):

bash
Copier le code
export OLLAMA_URL=http://localhost:11434
export OLLAMA_BASE_URL=http://localhost:11434

openclaw gateway --allow-unconfigured --bind lan
Canvas test page:

text
Copier le code
http://<VM_IP>:18793/__openclaw__/canvas/
If VirusTotal is not needed, related services can be removed from docker-compose.yml.

🔐 Hardening & production notes
For any serious or exposed deployment:

Replace all default passwords, tokens, and API keys

Store secrets in a vault or secrets manager

Use a reverse proxy (nginx / Traefik) with HTTPS

Restrict exposed ports via firewall

Monitor CPU, RAM, and disk usage (Elasticsearch + LLM are heavy)

🛣️ Roadmap
Add documented OpenCTI connectors:

MISP

AbuseIPDB

Publish CTI playbooks using OpenClaw + local LLM

Add:

Architecture diagram

Troubleshooting guide in docs/

Optional:

Full .env-based configuration refactor
