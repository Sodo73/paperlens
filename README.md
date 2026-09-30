# PaperLens

### Research Paper Assistant · Retrieval-Augmented Generation

**Maintained by Soheil Douzandeh · In development · Local infrastructure verified**

PaperLens is a research assistant project with a planned pipeline for ingesting arXiv papers, retrieving relevant passages, and generating answers grounded in research literature.

The current milestone verifies the local service stack, database connectivity, model inference, and workflow execution. Paper ingestion and end-to-end question answering are the next milestones.

## Verified Results

| Component | Purpose | Current evidence |
| --- | --- | --- |
| FastAPI | Application API | Health endpoint responded successfully; interactive documentation accessible |
| PostgreSQL | Relational storage for paper data | Notebook connected through host port 5433 |
| OpenSearch | Search index | Index health and mappings inspected; no papers indexed yet |
| OpenSearch Dashboards | Search administration | Container running |
| Redis | Cache infrastructure | Container healthy; application caching still to be verified |
| Ollama | Local language model inference | `llama3.2:1b` returned a response to a test prompt |
| Apache Airflow 2.10.3 | Workflow orchestration | Database and scheduler healthy; both tasks in the test DAG succeeded |

## My Contributions

- Configured and verified the Docker Compose environment on macOS.
- Resolved a PostgreSQL port conflict and updated the notebook connection settings.
- Adapted the notebook health check to the installed Airflow version and checked individual component status.
- Verified local model inference and manually triggered a successful Airflow workflow.
- Documented configuration changes, verification results, and steps for resuming the environment.

## Engineering Notes

### PostgreSQL Connectivity

A separate PostgreSQL installation was listening on the Mac's port 5432. Notebook connections reached that instance and reported that the expected database role did not exist.

Changing the Docker host mapping to `5433:5432` and connecting to `127.0.0.1:5433` resolved the issue. Connections between containers continue to use the database service's internal port 5432.

### Airflow Verification

The running container uses Airflow 2.10.3. I aligned the notebook check with its `/health` endpoint and inspected the database and scheduler fields rather than relying only on an HTTP success code.

The check returned:

```text
Database: healthy
Scheduler: healthy
```

A manually triggered DAG then completed both `hello_world` and `check_services` tasks.

### Build Recovery

The Airflow image build initially exhausted available storage. After freeing space, restarting Docker, and pruning unused build cache, the image built and the container started successfully.

The first build needs room for dependency downloads and intermediate image layers.

## Local Setup

Prerequisites:

- Docker Desktop with Docker Compose
- Python 3.12+
- `uv`

Run commands from the project root. The verified configuration is for local development.

For a new checkout, create `.env` from `.env.example` if it does not already exist and review its settings. Keep private configuration out of version control.

Install dependencies and start the verified services:

```bash
uv sync
docker compose up -d postgres redis opensearch ollama api opensearch-dashboards airflow
docker compose ps
```

The initial Airflow image build may take substantial time and disk space. This configuration publishes PostgreSQL on host port 5433.

Download the test model if needed:

```bash
docker exec rag-ollama ollama pull llama3.2:1b
```

Launch the verification notebooks:

```bash
uv run jupyter notebook
```

Open the setup notebook under `notebooks/` to inspect or repeat the checks.

### Local Interfaces

| Interface | Local address |
| --- | --- |
| FastAPI documentation | http://localhost:8000/docs |
| FastAPI health | http://localhost:8000/api/v1/health |
| Airflow dashboard | http://localhost:8080 |
| Airflow health | http://localhost:8080/health |
| OpenSearch Dashboards | http://localhost:5601 |
| Ollama API | http://localhost:11434 |
| PostgreSQL host connection | `127.0.0.1:5433` |

To pause the services while preserving their volumes:

```bash
docker compose stop
```

## Milestones

The checklist tracks my completed verification, regardless of whether supporting code is already present in the repository.

- [x] Verify service startup, database connectivity, and API health.
- [x] Confirm local language model inference.
- [x] Execute an Airflow test workflow and check scheduler health.
- [ ] Verify dependency compatibility and ingest arXiv papers.
- [ ] Validate keyword retrieval with indexed papers.
- [ ] Evaluate chunking, embeddings, and hybrid retrieval.
- [ ] Verify answers grounded in retrieved passages and streaming responses.
- [ ] Measure retrieval quality, latency, tracing, and cache behavior.
- [ ] Explore agentic retrieval and additional interfaces.

## MIT License and Attribution

Distributed under the [MIT License](LICENSE). Retain the original copyright and license notices in `LICENSE`.

PaperLens builds on the [JAM with AI RAG course repository](https://github.com/jamwithai/production-agentic-rag-course), created by **Shirin Khosravi Jam** and **Shantanu Ladhwe**, also shared as [arXiv Paper Curator](https://github.com/jamwithai/arxiv-paper-curator).

The base application architecture and course materials are their work. My contributions are the configuration changes, verification, troubleshooting, and documentation described above. 