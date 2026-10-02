# 🤖 HR Policy RAG Application

A production-ready Retrieval-Augmented Generation (RAG) application for answering questions about company HR policies. Built with LangChain, Groq LLM, Qdrant vector search, and deployed on AWS via GitHub Actions CI/CD pipeline.

**Live Demo:** [AWS Deployment Link](https://aws.amazon.com)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture & Components](#architecture--components)
- [Tech Stack & Gateway Integration](#tech-stack--gateway-integration)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Setup & Installation](#setup--installation)
- [Running the App](#running-the-app)
- [CI/CD Pipeline](#cicd-pipeline)
- [Environment Variables](#environment-variables)
- [Testing & Evaluation](#testing--evaluation)
- [Docker Deployment](#docker-deployment)
- [Contributing](#contributing)

---

## Overview

The HR Policy Assistant retrieves relevant information from HR policy documents using semantic search and answers employee questions with accuracy and grounding. It combines:

- **Document Ingestion**: Load and chunk HR policy documents
- **Vector Embeddings**: Convert text into searchable vectors using Jina
- **Cloud Vector Store**: Qdrant Cloud for scalable similarity search
- **LLM Gateway**: Portkey for provider routing, fallback, and cost optimization
- **Safety Guardrails**: Input/output validation to prevent misuse
- **Web UI**: Streamlit chat interface for end users
- **CLI Demo**: Command-line interface for testing
- **Evaluation**: LangSmith integration for answer quality assessment
- **Production Deployment**: Automated CI/CD with GitHub Actions → ECR → EC2

---

## Key Features

| Feature | Description |
|---------|-------------|
| **RAG Pipeline** | Document-grounded question answering with semantic retrieval |
| **Streamlit UI** | Real-time conversational chat interface |
| **CLI Demo** | Quick standalone testing with `python main.py` |
| **Vector Search** | Qdrant Cloud for fast semantic similarity matching |
| **LLM Gateway Routing** | Portkey integration for flexible LLM provider management |
| **Safety Guardrails** | Input/output filtering to ensure policy-compliant responses |
| **Evaluation Framework** | LangSmith integration for correctness and groundedness scoring |
| **Tracing & Monitoring** | LangSmith tracing for full request lifecycle visibility |
| **Docker Ready** | Dockerfile and Docker Compose for containerized execution |
| **Automated CI/CD** | GitHub Actions → Test → Evaluate → Build → Deploy to AWS ECR/EC2 |
| **Smoke Tests** | Pytest-based integration tests before build/deploy |
| **Multi-Provider LLM** | Gateway supports Groq, OpenAI, Gemini via Portkey |

---

## Architecture & Components

### RAG Pipeline Flow

```
HR Policy Document
        ↓
   Document Loader (TextLoader)
        ↓
   Text Splitter (500 chunk_size, 60 overlap)
        ↓
   Embeddings (Jina)
        ↓
   Vector Store (Qdrant Cloud)
        ↓
   Retriever (Top-3 results)
        ↓
   LLM Agent (Groq via Portkey)
        ↓
   Guardrails (Input/Output Checks)
        ↓
   Final Answer
```

### Core Modules

| Module | Purpose |
|--------|---------|
| `pipeline.py` | Orchestrates the entire RAG workflow (entry point) |
| `document_loader.py` | Loads HR policy text file using LangChain |
| `splitter.py` | Chunks document into overlapping segments |
| `embeddings.py` | Converts text to vectors using Jina API |
| `vector_store.py` | Manages Qdrant Cloud collection lifecycle |
| `gateway.py` | Routes LLM calls through Portkey (provider abstraction) |
| `llm.py` | Provides gateway-backed LLM instance |
| `agent.py` | Builds LangChain agent with tools and system prompt |
| `tools.py` | Defines search tool for the agent |
| `guardrails.py` | Checks input/output safety constraints |
| `evaluation.py` | Runs LangSmith evaluation on policy correctness |
| `tracing.py` | Configures LangSmith tracing for debugging |
| `config.py` | Centralized configuration and secrets management |
| `logger.py` | Structured logging across modules |

---

## Tech Stack & Gateway Integration

### Core Technologies

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **LLM Framework** | LangChain 0.1+ | Agent orchestration, tool binding, memory |
| **Language** | Python 3.11 | Core application language |
| **Vector DB** | Qdrant Cloud | Semantic search, embeddings storage |
| **Embeddings** | Jina AI (`jina-embeddings-v2-base-en`) | Text-to-vector conversion |
| **LLM Backend** | Groq (8B model) | Fast, cost-effective inference |
| **LLM Gateway** | Portkey AI | Provider routing, fallback, observability |
| **Web Framework** | Streamlit | Interactive chat UI, no frontend coding needed |
| **Container** | Docker | Reproducible environment |
| **Orchestration** | Docker Compose | Local multi-service setup |
| **Evaluation** | LangSmith | Quality metrics (correctness, groundedness) |
| **Tracing** | LangSmith Tracing | Request observability and debugging |
| **Testing** | Pytest | Smoke tests and integration tests |
| **CI/CD** | GitHub Actions | Automated test → evaluate → build → deploy |
| **Image Registry** | AWS ECR | Containerized app images |
| **Compute** | AWS EC2 | Production server deployment |

### LLM Gateway Features

The application uses **Portkey AI** as a gateway layer to abstract LLM providers and add production-grade capabilities:

| Feature | Capability | Benefit |
|---------|-----------|---------|
| **Provider Routing** | Route requests to Groq, OpenAI, Gemini, etc. | Switch providers without code changes |
| **API Key Management** | Store credentials behind provider "slugs" | Never expose raw API keys in code |
| **Provider Failover** | Configure fallback LLM if primary fails | High availability without manual retry logic |
| **Cost Optimization** | Route to cheaper providers based on latency/cost | Reduce LLM API spend |
| **Observability** | Log all LLM calls with latency & token usage | Monitor and debug at the gateway level |
| **OpenAI-Compatible Interface** | Use any provider like OpenAI (via `ChatOpenAI`) | Zero LangChain integration work |
| **Request Headers** | Use `x-portkey-provider` header for routing | Fine-grained control per request |

**Primary LLM Configuration:**
```yaml
Provider Slug:    @hrpolicyrag          (Groq, 8B model via Portkey)
Judge LLM Slug:   @hrpolicybackup       (Secondary Groq instance for evaluation)
Gateway URL:      PORTKEY_GATEWAY_URL   (OpenAI-compatible endpoint)
Authentication:   PORTKEY_API_KEY       (Long-lived gateway credential)
```

**How It Works:**
1. App calls Groq indirectly through Portkey gateway
2. API credentials stay in Portkey dashboard (not in code)
3. Gateway logs all calls for cost tracking and observability
4. Portkey can route to fallback provider if primary fails
5. Supports multi-provider setup (Groq primary, OpenAI backup, etc.)

---

## Project Structure

```
hr_policy_rag_application/
├── .github/
│   └── workflows/
│       └── deploy.yml              # CI/CD pipeline (test → eval → build → deploy)
├── .dockerignore                   # Files excluded from Docker image
├── .gitignore                       # Git ignore rules
├── Dockerfile                       # Container image definition
├── docker-compose.yml               # Local services (app + eval)
├── requirements.txt                 # Python dependencies
├── README.md                        # This file
│
├── app.py                           # Streamlit chat UI (entry: `streamlit run app.py`)
├── main.py                          # CLI demo (entry: `python main.py`)
├── evaluate.py                      # LangSmith evaluation runner
│
├── hr_assistant/                    # Core package
│   ├── __init__.py
│   ├── config.py                    # Environment variables & settings
│   ├── logger.py                    # Structured logging setup
│   ├── pipeline.py                  # Main RAG orchestration (entry point)
│   │
│   ├── document_loader.py           # Step 1: Load .txt documents
│   ├── splitter.py                  # Step 2: Split into chunks
│   ├── embeddings.py                # Step 3: Jina embeddings
│   ├── vector_store.py              # Step 4: Qdrant Cloud storage
│   │
│   ├── tools.py                     # Step 5: Search tool definition
│   ├── gateway.py                   # Step 6b: Portkey LLM gateway
│   ├── llm.py                       # Step 6: LLM initialization
│   ├── agent.py                     # Step 7: LangChain agent
│   │
│   ├── guardrails.py                # Input/output safety checks
│   ├── evaluation.py                # LangSmith evaluation suite
│   ├── tracing.py                   # LangSmith tracing setup
│
├── tests/
│   └── test_ask.py                  # Smoke test (integration check)
│
├── data/
│   └── hr_policy.txt                # HR policy document (required)
│       # Add your HR policy text file here
│
└── .env                             # Environment secrets (git-ignored)
    # GROQ_API_KEY=...
    # JINA_API_KEY=...
    # PORTKEY_API_KEY=...
    # QDRANT_URL=...
    # QDRANT_API_KEY=...
    # LANGSMITH_*=...
```

---

## Requirements

### Prerequisites

- **Python:** 3.10+
- **Docker & Docker Compose:** Optional (for containerized execution)
- **Git:** For cloning the repository

### API Keys & Credentials

You'll need to set up and provide:

1. **GROQ_API_KEY** – LLM inference (via Portkey)
   - Get it: https://console.groq.com

2. **JINA_API_KEY** – Text embeddings
   - Get it: https://jina.ai

3. **QDRANT_URL & QDRANT_API_KEY** – Vector store (cloud)
   - Get it: https://cloud.qdrant.io

4. **PORTKEY_API_KEY** – LLM gateway (provider routing)
   - Get it: https://portkey.ai
   - Set up provider slugs (`@hrpolicyrag`, `@hrpolicybackup`) in dashboard

5. **Optional – LangSmith Tracing:**
   - LANGSMITH_API_KEY
   - LANGSMITH_PROJECT
   - LANGSMITH_ENDPOINT

---

## Setup & Installation

### 1. Clone Repository

```bash
git clone https://github.com/dnyaneshwarmagar/hr_policy_rag_application.git
cd hr_policy_rag_application
```

### 2. Create Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate          # Linux/macOS
# .venv\Scripts\activate            # Windows
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

Or use `uv` (recommended for faster installs):

```bash
pip install uv
uv pip install -r requirements.txt
```

### 4. Create `.env` File

Copy the template below and fill in your credentials:

```bash
cat > .env << 'EOF'
# LLM Provider
GROQ_API_KEY=your_groq_api_key_here

# Embeddings
JINA_API_KEY=your_jina_api_key_here

# LLM Gateway (Portkey)
PORTKEY_API_KEY=your_portkey_api_key_here
PRIMARY_PROVIDER_SLUG=@hrpolicyrag
JUDDGE_PROVIDER_SLUG=@hrpolicybackup
JUDDGE_GROQ_API_KEY=your_backup_groq_key

# Vector Store (Qdrant Cloud)
QDRANT_URL=https://xxxxxx.qdrant.io
QDRANT_API_KEY=your_qdrant_api_key_here
QDRANT_COLLECTION_NAME=hr_policy

# Tracing (Optional)
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_API_KEY=your_langsmith_api_key_here
LANGSMITH_PROJECT=hr_policy_rag
EOF
```

### 5. Add HR Policy Document

Place your HR policy text file at:

```
data/hr_policy.txt
```

The file should contain the complete HR policy in plain text format.

### 6. Verify Setup

```bash
python -c "from hr_assistant.config import check_api_keys; check_api_keys()"
# If no error, you're good to go!
```

---

## Running the App

### Option 1: Streamlit Web UI

```bash
streamlit run app.py
```

Visit: `http://localhost:8501`

Features:
- Chat interface with message history
- Real-time policy lookups
- Streaming responses

### Option 2: CLI Demo

```bash
python main.py
```

Outputs:
- Question: "How many paid annual leave days do I get?"
- Answer: Retrieved from policy document

### Option 3: Docker Compose

```bash
docker compose up --build
```

Services:
- **app**: Streamlit UI on `http://localhost:8501`
- **eval**: Evaluation runner (optional, behind `profiles: ["tools"]`)

Run evaluation separately:

```bash
docker compose run eval
```

### Option 4: Docker (Standalone)

Build and run:

```bash
docker build -t hr-assistant:latest .
docker run -d \
  --name hr-assistant \
  -p 8501:8501 \
  --env-file .env \
  hr-assistant:latest
```

---

## CI/CD Pipeline

The project includes a **complete production-grade CI/CD pipeline** that automatically tests, evaluates, builds, and deploys the application to AWS.

### Pipeline Overview

```
GitHub Commit to main
        ↓
┌─────────────────────────────────────────┐
│  Job 1: TEST (Smoke Test)               │
│  - Pytest integration test              │
│  - Validates ask(agent, question)       │
│  - Fast (~2 min)                        │
└─────────────────────────────────────────┘
        ↓ (if passed)
┌─────────────────────────────────────────┐
│  Job 2: EVALUATE (LangSmith)            │
│  - Run 5 evaluation examples            │
│  - Check correctness & groundedness     │
│  - Fast evaluation (~3 min)             │
└─────────────────────────────────────────┘
        ↓ (if passed)
┌─────────────────────────────────────────┐
│  Job 3: BUILD & PUSH (ECR)              │
│  - Build Docker image                   │
│  - Tag with commit SHA + latest         │
│  - Push to AWS ECR                      │
│  - (~2 min)                             │
└─────────────────────────────────────────┘
        ↓ (if passed)
┌─────────────────────────────────────────┐
│  Job 4: DEPLOY (EC2)                    │
│  - SSH into EC2 instance                │
│  - Pull new image from ECR              │
│  - Stop old container, start new one    │
│  - Health check (60 second timeout)     │
│  - (~1 min)                             │
└─────────────────────────────────────────┘
        ↓
✅ DEPLOYED TO PRODUCTION
```

### Pipeline Configuration

**File:** `.github/workflows/deploy.yml`

**Trigger:**
- On every push to `main` branch
- Manual trigger via GitHub UI (`workflow_dispatch`)

**Concurrency:**
- Only one deployment at a time (prevents race conditions)
- Cancels in-progress runs if a new commit arrives

### Stage Details

#### 1. **TEST** – Smoke Test

```yaml
- Runs: pytest tests/
- Checks: ask(agent, question) returns a real answer
- Fails if: Test returns empty or crashes
- Duration: ~2 minutes
```

Command:
```bash
python -m pytest tests/ -v
```

#### 2. **EVALUATE** – LangSmith Quality Check

```yaml
- Runs: python evaluate.py
- Checks: 5 evaluation examples (subset for CI speed)
- Metrics: Correctness, Groundedness
- Fails if: Evaluation crashes or scores drop below threshold
- Duration: ~3 minutes
```

Command:
```bash
EVAL_LIMIT=5 python evaluate.py
```

#### 3. **BUILD & PUSH** – Docker → ECR

```yaml
- Builds Docker image from Dockerfile
- Tags: $ECR_REGISTRY/hr-assistant:$GITHUB_SHA
- Tags: $ECR_REGISTRY/hr-assistant:latest
- Pushes both tags to AWS ECR
- Duration: ~2 minutes
```

Commands:
```bash
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
docker build -t $IMAGE:$GITHUB_SHA -t $IMAGE:latest .
docker push $IMAGE:$GITHUB_SHA
docker push $IMAGE:latest
```

#### 4. **DEPLOY** – EC2 SSH Deployment

```yaml
- SSHes into EC2 instance
- Installs Docker + AWS CLI if needed (fresh instance support)
- Pulls new image from ECR
- Stops old container, starts new container
- Runs health check on Streamlit health endpoint
- Cleans up old images
- Duration: ~1 minute
```

Key Features:
- **Zero manual setup:** Fresh EC2 instances auto-configured
- **Health checks:** Waits up to 60s for Streamlit to be ready
- **Graceful stop:** Stops container safely, removes old images
- **Environment secrets:** `.env` file provisioned securely
- **Auto-restart:** Container has `--restart unless-stopped` policy

Deployment Script Highlights:
```bash
# Install Docker if needed
if ! command -v docker >/dev/null 2>&1; then
  sudo apt-get install -y docker.io
  sudo systemctl enable --now docker
fi

# Log in to ECR
aws ecr get-login-password --region $AWS_REGION | \
  sudo docker login --username AWS --password-stdin $ECR_REGISTRY

# Pull and restart
sudo docker pull $IMAGE
sudo docker stop $CONTAINER_NAME || true
sudo docker rm $CONTAINER_NAME || true
sudo docker run -d \
  --name $CONTAINER_NAME \
  --restart unless-stopped \
  -p 8501:8501 \
  --env-file $HOME/.env \
  $IMAGE

# Health check (loop 20 times, 3 sec interval = 60s total)
for i in $(seq 1 20); do
  if curl -fsS http://localhost:8501/_stcore/health >/dev/null; then
    echo "Deployed OK"
    exit 0
  fi
  sleep 3
done
```

### GitHub Secrets (Required)

Store these as GitHub repository secrets (Settings → Secrets and variables → Actions):

| Secret | Purpose |
|--------|---------|
| `APP_ENV_FILE` | Full `.env` file content (multi-line) for both test & deploy |
| `AWS_ACCESS_KEY_ID` | AWS IAM user credentials (long-lived) |
| `AWS_SECRET_ACCESS_KEY` | AWS IAM user credentials (long-lived) |
| `EC2_HOST` | EC2 public IP or hostname (e.g., `1.2.3.4`) |
| `EC2_USER` | EC2 SSH username (default: `ubuntu` for Ubuntu AMI) |
| `EC2_SSH_KEY` | EC2 private SSH key (PEM format, multi-line) |

### Pipeline Monitoring

**View pipeline runs:**
1. Go to your GitHub repo
2. Click **Actions** tab
3. Click **deploy** workflow
4. View real-time logs for each job

**Common Failure Scenarios:**

| Scenario | Fix |
|----------|-----|
| **Test fails** | Policy document missing or app has bugs; check logs |
| **Eval fails** | Lower eval scores; check answer quality in LangSmith |
| **ECR push fails** | AWS credentials missing/expired; refresh GitHub secrets |
| **EC2 deploy fails** | SSH key wrong, instance down, or Streamlit port 8501 blocked |
| **Health check fails** | Streamlit crashed on startup; check container logs on EC2 |

### AWS Architecture

```
GitHub Repo (main branch)
        ↓
GitHub Actions (runner: ubuntu-latest)
        ↓
AWS ECR (Elastic Container Registry)
    └─ Image: hr-assistant:$GITHUB_SHA
    └─ Image: hr-assistant:latest
        ↓
AWS EC2 (c5.large, us-east-1)
    └─ Running Docker container
    └─ Port 8501 exposed
    └─ .env file mounted
```

**Infrastructure Diagram:**
```
┌──────────────────────────┐
│     EC2 Instance         │
│  (us-east-1)            │
│ ┌────────────────────┐   │
│ │  Docker Container  │   │
│ │  hr-assistant:xyz  │   │
│ │ ┌────────────────┐ │   │
│ │ │ Streamlit App  │ │   │
│ │ │ :8501          │ │   │
│ │ └────────────────┘ │   │
│ │ .env (mounted)     │   │
│ └────────────────────┘   │
│ Security Group: 8501     │
└──────────────────────────┘
        ↑
   AWS ECR
   (Image Store)
        ↑
 GitHub Actions
   (CI/CD)
        ↑
   GitHub Repo
  (main branch)
```

### Customizing the Pipeline

**Modify pipeline stages:**

Edit `.github/workflows/deploy.yml`:

```yaml
# Change evaluation sample size
env:
  EVAL_LIMIT: "10"  # Run 10 examples instead of 5

# Change AWS region
env:
  AWS_REGION: us-west-2

# Change container port
env:
  APP_PORT: 8080
```

**Disable deployment (test only):**

Remove or comment out the `deploy` job in the workflow file.

**Add approval step before deploy:**

Add to the deploy job:
```yaml
environment:
  name: production
  required_reviewers:
    - your-github-username
```

---

## Environment Variables

### Required Variables

```bash
# LLM Provider (Groq)
GROQ_API_KEY=gsk_xxxxxxxxxxxxx

# Embeddings (Jina)
JINA_API_KEY=jina_xxxxxxxxxxxxx

# LLM Gateway (Portkey)
PORTKEY_API_KEY=pk_xxxxxxxxxxxxx
PRIMARY_PROVIDER_SLUG=@hrpolicyrag
JUDDGE_PROVIDER_SLUG=@hrpolicybackup
JUDDGE_GROQ_API_KEY=gsk_xxxxxxxxxxxxx

# Vector Store (Qdrant Cloud)
QDRANT_URL=https://xxxxxxx.qdrant.io
QDRANT_API_KEY=qdrant_xxxxxxxxxxxxx
QDRANT_COLLECTION_NAME=hr_policy
```

### Optional Variables

```bash
# Tracing & Evaluation (LangSmith)
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_API_KEY=ls_xxxxxxxxxxxxx
LANGSMITH_PROJECT=hr_policy_rag

# Model Configuration
LLM_MODEL_NAME=openai/gpt-oss-20b
EMBEDDING_MODEL_NAME=jina-embeddings-v2-base-en
CHUNK_SIZE=500
CHUNK_OVERLAP=60
TOP_K_RESULTS=3
```

### Loading from `.env`

The app automatically loads from `.env` file using `python-dotenv`:

```python
from dotenv import load_dotenv
load_dotenv()
```

---

## Testing & Evaluation

### Unit & Integration Tests

**Run all tests:**

```bash
pytest tests/ -v
```

**Test file:** `tests/test_ask.py`

**What it tests:**
- Builds real agent (with real API calls)
- Asks a sample question
- Validates response is non-empty string

This is a **smoke test**, not a mocked test — it validates end-to-end functionality.

### LangSmith Evaluation

**Run evaluation suite (full):**

```bash
python evaluate.py
```

**Run subset for CI:**

```bash
EVAL_LIMIT=5 python evaluate.py
```

**What it evaluates:**
- Correctness: Does the answer match the expected response?
- Groundedness: Is the answer supported by the retrieved documents?
- Uses LangSmith evaluator framework

**View results:**
1. Go to https://smith.langchain.com
2. Open your LANGSMITH_PROJECT
3. View "Experiments" → latest run

---

## Docker Deployment

### Build Image Locally

```bash
docker build -t hr-assistant:latest .
```

### Run Container

```bash
docker run -d \
  --name hr-assistant \
  -p 8501:8501 \
  --env-file .env \
  hr-assistant:latest
```

### Push to ECR

```bash
# Get ECR login token
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com

# Tag and push
docker tag hr-assistant:latest $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/hr-assistant:latest
docker push $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/hr-assistant:latest
```

### Dockerfile Overview

```dockerfile
FROM python:3.11-slim

# Use uv for fast dependency installation
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app

COPY requirements.txt .
RUN uv pip install --system --no-cache -r requirements.txt

COPY . .

EXPOSE 8501

CMD ["streamlit", "run", "app.py", "--server.address=0.0.0.0"]
```

**Key points:**
- Python 3.11 slim image (small size)
- Uses `uv` for fast dependency installation
- Exposes port 8501 (Streamlit default)
- Runs Streamlit on 0.0.0.0 (accessible from host)

---

## Common Use Cases

### 1. **Add New Policy Documents**

Replace `data/hr_policy.txt` with your document and rebuild:

```bash
# Clear existing vector store
# (delete collection in Qdrant Cloud dashboard, or change QDRANT_COLLECTION_NAME)

# Re-run the app to ingest new document
python main.py
```

### 2. **Add Evaluation Examples**

Edit `hr_assistant/evaluation.py` to add more test cases in the evaluation dataset.

### 3. **Switch LLM Providers**

Edit Portkey dashboard to change `@hrpolicyrag` slug target (Groq → OpenAI → Gemini, etc.). No code changes needed!

### 4. **Adjust Chunk Size**

In `hr_assistant/config.py`:

```python
CHUNK_SIZE = 1000      # Longer chunks
CHUNK_OVERLAP = 100    # More overlap
TOP_K_RESULTS = 5      # Return more results
```

Then rebuild vector store.

### 5. **Enable LangSmith Tracing**

In `.env`:

```bash
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=ls_xxxxx
LANGSMITH_PROJECT=hr_policy_rag
```

All requests will now appear in LangSmith dashboard.

---

## Troubleshooting

### "Missing GROQ_API_KEY"

→ Add `GROQ_API_KEY` to `.env` file and re-run

### "Qdrant Cloud collection not found"

→ Run `python main.py` once to create the collection

### "Embedding failed (Jina error)"

→ Check `JINA_API_KEY` is correct and quota not exceeded

### "Portkey gateway rejected request"

→ Verify `PORTKEY_API_KEY` and provider slugs exist in Portkey dashboard

### "Health check failed after 60s" (in CI/CD)

→ Streamlit didn't start on EC2; SSH in and check: `docker logs hr-assistant`

### "Docker image push failed"

→ Verify AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, and ECR repo exists

---

## Architecture Decisions

| Decision | Rationale |
|----------|-----------|
| **Qdrant Cloud** (not in-memory FAISS) | Scalable, persistent, multi-user support |
| **Jina Embeddings** | Fast, multilingual, good cost/quality ratio |
| **Portkey Gateway** | Vendor-agnostic, cost control, observability |
| **Streamlit UI** | Rapid iteration, no frontend code needed |
| **LangChain Agents** | Tool binding, memory, structured prompts |
| **GitHub Actions + ECR + EC2** | Fully automated, GitHub-native, cost-effective |
| **Docker** | Reproducible, portable across environments |

---

## Performance Notes

- **Retrieval:** ~300ms (Qdrant search)
- **Embedding:** ~500ms (Jina API call)
- **LLM Inference:** ~1-2s (Groq via Portkey)
- **Total Response:** ~2-3 seconds

**Bottleneck:** LLM inference time (unavoidable for quality)

**Optimization ideas:**
- Cache embeddings for frequently asked questions
- Use smaller LLM for simple queries
- Implement response streaming in UI

---

## License

This project does not include a license file. If publishing publicly, consider adding MIT or Apache 2.0.

---

## Contributing

Contributions welcome! Areas for improvement:

- [ ] Add more evaluation examples
- [ ] Improve retrieval ranking
- [ ] Add more guardrail rules
- [ ] Implement response caching
- [ ] Add cost tracking dashboard
- [ ] Multi-document support
- [ ] User authentication
- [ ] Rate limiting

---

## Support & Contact

For questions or issues:

1. **Check existing issues** on GitHub
2. **Review docs** in project (linked in workflow comments)
3. **Contact maintainer** for production deployment help

---

## Deployment Status

| Environment | Status | URL |
|-------------|--------|-----|
| **CI/CD** | ![CI/CD Status](https://github.com/dnyaneshwarmagar/hr_policy_rag_application/actions/workflows/deploy.yml/badge.svg?branch=main) | [View Workflow](https://github.com/dnyaneshwarmagar/hr_policy_rag_application/actions) |
| **Production** | ✅ Deployed | [Live Demo](http://ec2-54-80-123-123.compute-1.amazonaws.com:8501/) |

---

*
**Repository:** [dnyaneshwarmagar/hr_policy_rag_application](https://github.com/dnyaneshwarmagar/hr_policy_rag_application)
