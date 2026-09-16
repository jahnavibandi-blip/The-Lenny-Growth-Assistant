# The-Lenny-Growth-Assistant
AI-powered conversational assistant for Lenny's Podcast transcripts. The application provides grounded product and growth Q&A, session-aware conversations, Ship 30 for 30-style content generation, and Markdown/HTML artifact generation with an in-app Artifact Viewer.

Features

Grounded conversational Q&A over Lenny's Podcast transcripts

Source-aware answers grounded in the knowledge base

Independent chat sessions with PostgreSQL persistence

FastAPI backend

Agent-based routing and specialized content-generation skill

Ship 30 for 30-style essay generation

Markdown and HTML/CSS artifact generation

In-app Artifact Viewer

Local LLM support through Ollama

Configurable cloud LLM support

Automated tests for critical backend behavior

Docker-based reproducible setup

Structured logging and graceful error handling

Architecture Overview

The application follows a full-stack architecture:

                    +----------------------+
                    |      Frontend        |
                    |   Chat + Artifacts   |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |       FastAPI        |
                    |      Backend API     |
                    +----------+-----------+
                               |
             +-----------------+-----------------+
             |                 |                 |
             v                 v                 v
      +-------------+   +-------------+   +-------------+
      | PostgreSQL  |   | RAG /       |   | Agent /     |
      | Sessions &  |   | Knowledge   |   | LLM Router  |
      | Messages    |   | Base        |   |             |
      +-------------+   +------+------+   +------+------+
                               |                 |
                               v                 v
                        Lenny Transcripts   +----+----+
                                           |         |
                                           v         v
                                        Ollama   Cloud LLM

Main Components

Frontend

Conversational chat interface

Session management

Source display

Artifact Viewer for generated Markdown/HTML

FastAPI Backend

REST API

Request validation

Session handling

Agent routing

RAG orchestration

Artifact generation

Health checks and error handling

PostgreSQL

Stores session IDs

Stores conversations/messages

Stores timestamps and user metadata

Knowledge Base

Loads Lenny's Podcast transcript content

Chunks/selects and indexes transcript content

Retrieves relevant passages for user questions

Keeps source information so answers can be traced back to transcripts

Agent / LLM Layer

Routes requests to the configured model

Supports local Ollama for the demo

Supports a cloud LLM provider

Provides specialized content-generation behavior

Prerequisites

Install the following before running the application:

Git

Docker and Docker Compose

Python 3.11+ if running the backend outside Docker

Node.js 18+ if running the frontend outside Docker

Ollama for the mandatory local-model demo

A compatible local Ollama model

PostgreSQL if running without Docker

A cloud LLM API key if cloud-model support is enabled

Use the exact versions required by the project's requirements.txt and package.json if they differ from the versions above.

Installation

1. Clone the repository

git clone <YOUR_GITHUB_REPOSITORY_URL>
cd lenny-growth-assistant

2. Create the environment file

cp .env.example .env

On Windows PowerShell:

Copy-Item .env.example .env

Fill in only the variables required by your configuration. Never commit .env.

3. Start the application with Docker Compose

docker compose up --build

To run in detached mode:

docker compose up --build -d

4. Stop the application

docker compose down

To remove local database volumes as well:

docker compose down -v

Use the volume-removal command only when you intentionally want to reset persisted local data.

Environment Variables

Create .env from .env.example.

Example:

DATABASE_URL=postgresql://<user>:<password>@<host>:5432/<database>

LLM_PROVIDER=ollama

OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=<your-ollama-model>

ANTHROPIC_API_KEY=
OPENAI_API_KEY=

CLOUD_LLM_PROVIDER=
CLOUD_LLM_MODEL=

Variable descriptions

Variable

Required

Purpose

DATABASE_URL

Yes

PostgreSQL connection string

LLM_PROVIDER

Yes

Selects the active LLM provider

OLLAMA_BASE_URL

For Ollama

Ollama server URL

OLLAMA_MODEL

For Ollama

Local model used by the application

ANTHROPIC_API_KEY

If Anthropic is enabled

Cloud Anthropic authentication

OPENAI_API_KEY

If OpenAI is enabled

Cloud OpenAI authentication

CLOUD_LLM_PROVIDER

If cloud mode is enabled

Cloud provider selection

CLOUD_LLM_MODEL

If cloud mode is enabled

Cloud model selection

Keep secrets only in .env or the deployment environment. Do not commit API keys.

Local Model Setup with Ollama

Ollama is used for the local-model demonstration.

1. Install Ollama

Install Ollama for your operating system from the official Ollama website.

2. Start Ollama

ollama serve

If Ollama is already running as a background service, this command may not be necessary.

3. Pull a model

Use a model that runs comfortably on your machine:

ollama pull <MODEL_NAME>

For example, after selecting a supported model:

ollama run <MODEL_NAME>

4. Configure the application

Set:

LLM_PROVIDER=ollama
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=<MODEL_NAME>

5. Verify Ollama

ollama list

The application should handle Ollama being unavailable gracefully and report a useful error rather than crashing.

Cloud Model Setup

The application also supports a cloud LLM provider through the model configuration layer.

Obtain an API key from the cloud provider used by this project.

Put the key in .env.

Select the cloud provider/model through the application's configuration.

Restart the application.

Example:

LLM_PROVIDER=cloud
CLOUD_LLM_PROVIDER=<provider>
CLOUD_LLM_MODEL=<model>
ANTHROPIC_API_KEY=<your-key>

Use only the variables required by the provider implemented in this repository.

Model Toggle

The active provider should be visible through the application's configuration or UI.

Typical configuration:

LLM_PROVIDER=ollama

or:

LLM_PROVIDER=cloud

The provider abstraction keeps application logic independent from the underlying model.

If the selected provider is unavailable, the application should return a clear error and follow the documented fallback behavior rather than silently producing an ungrounded answer.

Knowledge Base / RAG

The assistant uses Lenny's Podcast transcripts as its knowledge source.

The ingestion pipeline should:

Load transcript content.

Clean and normalize the text.

Split content into retrieval-friendly chunks.

Index the chunks.

Preserve transcript/source metadata.

Retrieve relevant passages for each user query.

Pass retrieved context to the agent/LLM.

Identify the relevant source in the final response.

If the retrieved material does not support an answer, the assistant should acknowledge that the available material is insufficient rather than inventing an answer.

Running the Application

Docker

docker compose up --build

Open the frontend URL shown by the Docker Compose configuration.

Backend only

If the backend is configured for local execution:

cd backend
python -m venv .venv

Activate the virtual environment.

Windows:

.venv\Scripts\Activate.ps1

macOS/Linux:

source .venv/bin/activate

Install dependencies:

pip install -r requirements.txt

Start FastAPI:

uvicorn app.main:app --reload

Frontend only

If the frontend is configured for local execution:

cd frontend
npm install
npm run dev

Use the API URL configured by the frontend environment.

API Health Check

The backend should expose a health endpoint.

Example:

curl http://localhost:8000/health

The exact host, port, and endpoint should match the implementation in this repository.

Testing

Run the automated backend tests:

pytest

The test suite should cover critical behavior such as:

API request/response validation

Health endpoints

Chat/session behavior

Retrieval behavior

Model routing

PostgreSQL persistence

Error handling

Manual UI Test Plan

Open the application.

Start a new chat.

Ask a product/growth question.

Verify that the response is grounded in transcript content.

Verify that relevant sources are shown.

Ask a follow-up question and verify session context is preserved.

Ask a question that is not supported by the knowledge base and verify the assistant acknowledges insufficient evidence.

Request a Ship 30 for 30-style essay.

Request a Markdown artifact.

Request an HTML/CSS artifact.

Verify the Artifact Viewer renders the result in-app.

Switch to Ollama/local model mode and repeat a basic query.

Test an unavailable model/API configuration and verify graceful error handling.

Create a new chat and verify that its context is independent.

Artifact Viewer Security

Generated HTML is treated as untrusted content.

The Artifact Viewer should use an isolation or sanitization strategy appropriate to the implementation. The goal is to prevent generated HTML from accessing privileged application data or executing unsafe behavior.

Document the exact controls implemented in architecture.md, including:

What HTML is allowed to render

What scripts or capabilities are blocked

Whether an iframe sandbox is used

What sanitization is performed

Why the chosen approach is appropriate

Troubleshooting

Ollama is not available

Check that Ollama is running:

ollama list

If necessary:

ollama serve

Also verify:

OLLAMA_BASE_URL=http://localhost:11434

and confirm that the configured model exists.

Model not found

List installed models:

ollama list

Pull the configured model:

ollama pull <MODEL_NAME>

Make sure OLLAMA_MODEL exactly matches the installed model name.

Database connection failure

Check:

PostgreSQL is running.

DATABASE_URL is correct.

The database exists.

Username and password are correct.

The Docker Compose database service is healthy when using Docker.

Cloud LLM authentication failure

Check:

The correct API key is configured.

The selected provider matches the configured key.

The key is not expired or revoked.

.env is loaded by the backend.

Never place the API key directly in source code.

Retrieval returns no useful results

Check:

Transcript data was successfully ingested.

The index/vector store is available.

Chunking completed successfully.

The retrieval query is being generated correctly.

Transcript source metadata is preserved.

Frontend cannot connect to backend

Check:

Backend is running.

The frontend API base URL is correct.

The configured port is correct.

Docker services can communicate with each other.

Browser/network errors in the frontend console.

Tests fail

Run:

pytest -v

Then inspect the first failing test and its traceback. Verify that required environment variables, database services, and local model services are available.

Project Structure

lenny-growth-assistant/
├── backend/
├── frontend/
├── data/
├── agents/
├── docs/
│   ├── PRD.md
│   ├── design.md
│   └── architecture.md
├── .env.example
├── .gitignore
├── docker-compose.yml
└── README.md

The exact structure may vary based on implementation.

Documentation

docs/PRD.md — discovery brief, requirements, assumptions, scope, risks, acceptance criteria, and implementation plan.

docs/design.md — UI/UX principles, information architecture, interaction states, responsive behavior, accessibility, and design decisions.

docs/architecture.md — database schema, API endpoints, component boundaries, RAG flow, agent routing, model configuration, security, and deployment topology.

agents/ — coding-agent transcripts/logs, including relevant failed attempts and corrections.

Demo

Add the final 2–3 minute YouTube demo link here:

https://www.youtube.com/<YOUR_VIDEO>

The demo should show the problem, working product, local Ollama demonstration, and one important technical trade-off.

Security

Never commit .env.

Never commit API keys, passwords, tokens, or database credentials.

Treat generated HTML as untrusted.

Validate API inputs.

Handle model, retrieval, database, and artifact-rendering failures gracefully.

Known Limitations

Document any limitations specific to the implementation here. Examples may include local-model quality, transcript coverage, retrieval edge cases, model latency, or unsupported artifact features.

