# HoneyBee AI Usage Report

## Executive Summary

HoneyBee is an **LLM-powered security testing tool** that generates intentionally misconfigured application deployments. It uses AI to create Dockerfiles, Docker Compose files, and Nuclei security scanning templates.

---

## 1. AI Providers Supported

| Provider | Configuration |
|----------|---------------|
| **OpenAI** | API key via `OPENAI_API_KEY` env var |
| **Azure OpenAI** | API key + endpoint, API version `2024-07-01-preview` |
| **Jina AI** | Secondary service for web content extraction |

**Default Model**: `gpt-4o` (configurable via UI)

**Code Location**: `honeybee/src/client_provider.py`

---

## 2. Core AI Functions

Located in [`honeybee/src/generators.py`](../honeybee/src/generators.py):

| Function | Purpose |
|----------|---------|
| [`generate_dockerfile()`](../honeybee/src/generators.py#L33-L40) | Creates Dockerfiles with security misconfigurations |
| [`generate_docker_compose()`](../honeybee/src/generators.py#L43-L50) | Creates multi-service Docker Compose configs |
| [`generate_nuclei()`](../honeybee/src/generators.py#L53-L68) | Creates Nuclei security scanning templates |
| [`extract_application_from_markdown()`](../honeybee/src/generators.py#L121-L133) | Parses blog posts to extract app names |
| [`generate_from_prompt()`](../honeybee/src/generators.py#L8-L30) | Generic LLM call wrapper with retry logic |

---

## 3. System Prompts

Located in `honeybee/prompts/`:

| Prompt File | Role |
|-------------|------|
| [`generate_dockerfile.md`](../honeybee/prompts/generate_dockerfile.md) | DockerfileGenBot - containerization expert |
| [`generate_dockercompose.md`](../honeybee/prompts/generate_dockercompose.md) | DockerComposeGenBot - orchestration expert |
| [`write_nuclei_rule.md`](../honeybee/prompts/write_nuclei_rule.md) | Nuclei AI expert (~68KB of instructions) |
| [`extract_application.md`](../honeybee/prompts/extract_application.md) | Cloud security specialist for parsing |
| [`analyze_misconfig.md`](../honeybee/prompts/analyze_misconfig.md) | Misconfiguration analysis |
| [`find_misconfig.md`](../honeybee/prompts/find_misconfig.md) | Misconfiguration discovery |
| [`list_misconfig.md`](../honeybee/prompts/list_misconfig.md) | Catalog generation |

---

## 4. Input Modes (AI-Powered)

The main app (`honeybee/1_🐝_app.py`) supports 4 input modes:

1. **Browse Catalog** - Select predefined misconfigurations
2. **Custom Input** - Manual app name + misconfiguration
3. **Import from URL** - Uses **Jina AI** to fetch web content, convert to markdown, then LLM extracts app name
4. **Paste Content** - Manual markdown input processed by LLM

---

## 5. UI Components with AI Integration

| Tab File | AI Action |
|----------|-----------|
| `tabs/dockerfile_tab.py` | "Generate Dockerfile" button triggers LLM call |
| `tabs/dockercompose_tab.py` | "Generate Docker Compose" button triggers LLM call |
| `tabs/nuclei_tab.py` | "Generate Nuclei" button triggers LLM call |

---

## 6. LLM API Call Pattern

```python
chat_completion = client.chat.completions.create(
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt},
    ],
    model=model,
)
```

- **Retry logic**: Up to 5 attempts for JSON parsing failures
- **Output formats**: JSON (wrapped in markdown code blocks) or raw YAML

---

## 7. Query History & Persistence

Located in `honeybee/src/query_history.py`:

- All LLM interactions saved to `.qhistory/` directory
- JSON format with timestamps
- Viewable via History page (`pages/1_⏳_history.py`)

---

## 8. What's NOT Used

- Vector databases
- Embeddings
- RAG (Retrieval-Augmented Generation)
- Local ML models
- Fine-tuned models
- Agents/autonomous systems

---

## 9. Architecture Flow

```
User Input → Input Mode Selection
                    ↓
         [Jina AI for URL mode]
                    ↓
    System Prompt + User Prompt
                    ↓
       OpenAI/Azure OpenAI API
                    ↓
      Parse Response (JSON/YAML)
                    ↓
    Display + Save to Query History
```

---

## 10. Key Takeaways

1. **Single LLM dependency** - OpenAI SDK (`openai==1.66.2`) for all AI features
2. **Prompt engineering focused** - 7 specialized system prompts
3. **Generation-only AI** - No classification, embedding, or analysis tasks
4. **Enterprise-ready** - Azure OpenAI support for corporate environments
5. **Web ingestion** - Jina AI enables learning from security blog posts

---

## 11. Dependencies

From `requirements.txt`:

```
openai==1.66.2      # Primary AI dependency
PyYAML==6.0.2       # YAML parsing for outputs
Requests==2.32.3    # HTTP requests (Jina AI calls)
streamlit==1.46.0   # Web UI framework
```

---

## 12. Environment Variables

| Variable | Purpose |
|----------|---------|
| `OPENAI_API_KEY` | OpenAI API authentication |
| `AZURE_OPENAI_API_KEY` | Azure OpenAI authentication |
| `AZURE_OPENAI_ENDPOINT` | Azure deployment endpoint |
| `JINA_API_TOKEN` | Web content extraction service |
