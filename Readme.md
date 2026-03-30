<div align="center">

# 🧠 QuizMind-AI — Intelligent Quiz Generator

**An AI-powered quiz generation and evaluation app built with Streamlit.**  
Pick any topic, any difficulty — get an instant personalized quiz. Deployed on AWS EC2 with Docker.

[🚀 Quick Start](#-quick-start-local) · [✨ Features](#-features) · [📁 Project Structure](#-project-structure) · [🐳 Docker](#-docker-deployment) · [☁️ AWS EC2](#-aws-ec2-deployment)

---

</div>

## 📌 Overview

**QuizMind-AI** is a fully interactive quiz application powered by a Large Language Model. It dynamically generates quizzes on **any topic** in two formats — Multiple Choice and Fill in the Blank — across three difficulty levels. Users attempt the quiz in-app, receive instant feedback, and can download results as a CSV.

---

## ✨ Features

| Feature | Description |
|--------|-------------|
| 🤖 **AI Question Generation** | LLM generates unique, contextual questions for any topic |
| 📝 **Two Question Types** | Multiple Choice and Fill in the Blank |
| 🎯 **Three Difficulty Levels** | Easy, Medium, Hard |
| ⚡ **Instant Evaluation** | Automatic scoring and per-question feedback |
| 📊 **Results Dashboard** | Score %, ✅/❌ breakdown with correct answers shown |
| 💾 **CSV Export** | Download full results with answers and correctness flags |
| 🐳 **Docker Ready** | Containerized for consistent deployments |
| ☁️ **AWS EC2 Hosted** | Accessible from anywhere via public IP |

---

## 📁 Project Structure

```
quizmind-ai/
│
├── application.py                  # Streamlit entry point
│
├── src/
│   ├── __init__.py
│   ├── common/                     # Shared utilities and base classes
│   ├── config/                     # App configuration and settings
│   ├── generator/                  # Quiz question generation logic
│   ├── llm/                        # LLM client setup and API wrappers
│   ├── models/                     # Data models / schema definitions
│   ├── prompts/                    # Prompt templates for the LLM
│   └── utils/                      # Helper functions and QuizManager
│
├── logs/                           # Auto-generated application logs
├── venv/                           # Virtual environment (not committed)
│
├── .env                            # API keys — never commit this
├── .gitignore
├── Dockerfile                      # Docker container definition
├── requirements.txt
├── setup.py
└── README.md
```

---

## 🚀 Quick Start (Local)

### 1. Clone the Repository

```bash
git clone https://github.com/----
cd quizmind-ai
```

### 2. Create a Virtual Environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
# or using setup.py
pip install -e .
```

### 4. Configure Environment Variables

```bash
# Copy the example and fill in your key
cp .env.example .env
nano .env
```

```env
# Add whichever LLM provider you are using
GROQ_API_KEY=...
.
```

### 5. Run the App

```bash
streamlit run application.py
```

Visit [http://localhost:8501](http://localhost:8501)

---

## 🐳 Docker Deployment

> Docker is already configured via the `Dockerfile` in the root of the project.

### Build the Image

```bash
docker build -t quizmind-ai .
```

### Run the Container

```bash
docker run -p 8501:8501 --env-file .env quizmind-ai
```

Visit [http://localhost:8501](http://localhost:8501)

### Run in Detached Mode (Background)

```bash
docker run -d -p 8501:8501 --env-file .env --name quizmind quizmind-ai
```

### Useful Docker Commands

```bash
# View running containers
docker ps

# View logs
docker logs -f quizmind

# Stop the container
docker stop quizmind

# Remove the container
docker rm quizmind

# Rebuild after code changes
docker build -t quizmind-ai . && docker run -d -p 8501:8501 --env-file .env --name quizmind quizmind-ai
```

---

## ☁️ AWS EC2 Deployment

### Architecture

```
Users (Browser)
      │
      │  HTTP :8501
      ▼
┌───────────────────────────┐
│         AWS EC2           │
│    Ubuntu 22.04 LTS       │
│                           │
│  ┌─────────────────────┐  │
│  │   Docker Container  │  │
│  │   quizmind-ai       │  │
│  │                     │  │
│  │  application.py     │  │
│  │  (Streamlit :8501)  │  │
│  └─────────────────────┘  │
└───────────────────────────┘
      │
      │  HTTPS API calls
      ▼
   LLM Provider API
```

---

### Step 1 — Launch an EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instance**
2. Choose **Ubuntu Server 22.04 LTS** (Free Tier eligible)
3. Select instance type — **t2.micro** (free tier) or **t2.small** (recommended)
4. Create or select a **Key Pair** — download the `.pem` file safely
5. Under **Security Group**, add these inbound rules:

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | 22 | My IP |
| Custom TCP | TCP | 8501 | 0.0.0.0/0 |

6. Click **Launch Instance**

---

### Step 2 — Connect to EC2

```bash
# Set correct permissions on key file
chmod 400 your-key.pem

# SSH into the instance
ssh -i "your-key.pem" ubuntu@<your-ec2-public-ip>
```

---

### Step 3 — Install Docker on EC2

```bash
# Update packages
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install docker.io -y

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add ubuntu user to docker group (avoid using sudo every time)
sudo usermod -aG docker ubuntu

# Apply group change without logout
newgrp docker

# Verify Docker works
docker --version
```

---

### Step 4 — Clone the Repository

```bash
sudo apt install git -y
git clone https://github.com/------
cd quizmind-ai
```

---

### Step 5 — Configure Environment Variables

```bash
nano .env
```

```env
GROQ_API_KEY=...
# or whichever LLM provider you use
```

Save: `Ctrl+O` → `Enter` → `Ctrl+X`

---

### Step 6 — Build and Run with Docker

```bash
# Build the image
docker build -t quizmind-ai .

# Run the container (detached, always restarts)
docker run -d \
  -p 8501:8501 \
  --env-file .env \
  --name quizmind \
  --restart always \
  quizmind-ai
```

The `--restart always` flag ensures the app automatically restarts if EC2 reboots.

---

### Step 7 — Access Your App

```
http://<your-ec2-public-ip>:8501
```

Find your public IP in **AWS Console → EC2 → Instances → Public IPv4 address**

---

### Step 8 — Update the App (Pull + Redeploy)

Whenever you push new code:

```bash
cd quizmind-ai

# Pull latest changes
git pull

# Rebuild and restart container
docker stop quizmind
docker rm quizmind
docker build -t quizmind-ai .
docker run -d -p 8501:8501 --env-file .env --name quizmind --restart always quizmind-ai
```

---

### Useful EC2 + Docker Commands

```bash
# View running containers
docker ps

# Live logs from the app
docker logs -f quizmind

# Check app is on port 8501
sudo lsof -i :8501

# Restart container
docker restart quizmind

# Stop and remove
docker stop quizmind && docker rm quizmind
```

---

### EC2 Cost Estimate

| Instance | vCPU | RAM | Est. Monthly Cost |
|----------|------|-----|-------------------|
| t2.micro | 1 | 1 GB | Free tier / ~$8.50 |
| t2.small | 1 | 2 GB | ~$17/month |
| t2.medium | 2 | 4 GB | ~$34/month |

> ⚠️ **Stop or terminate your instance when not in use to avoid unexpected AWS charges.**

---

## 🔑 Key Modules

| Module | Path | Purpose |
|--------|------|---------|
| Entry point | `application.py` | Streamlit UI + session state management |
| LLM client | `src/llm/` | API connection and model calls |
| Prompt templates | `src/prompts/` | Structured prompts sent to the LLM |
| Question generator | `src/generator/` | Builds and parses quiz questions |
| Data models | `src/models/` | Question and result schema definitions |
| Quiz manager | `src/utils/` | Generate, attempt, evaluate, export |
| Config | `src/config/` | App-wide settings and constants |
| Common | `src/common/` | Shared base classes and utilities |
| Logs | `logs/` | Auto-generated runtime logs |

---

## 🗂️ CSV Export Format

| Column | Type | Description |
|--------|------|-------------|
| `question_number` | int | 1-indexed position |
| `question` | str | Question text |
| `user_answer` | str | What the user selected or entered |
| `correct_answer` | str | Correct answer from the model |
| `is_correct` | bool | True if answers match |

---

## 📦 Requirements

```txt
streamlit
python-dotenv
pandas
# your LLM SDK — e.g. openai / anthropic / google-generativeai
```

Install:

```bash
pip install -r requirements.txt
```

---

<div align="center">

Built with ❤️ using Streamlit · Containerized with 🐳 Docker · Deployed on ☁️ AWS EC2

⭐ **Star this repo if it helped you!** ⭐

</div>