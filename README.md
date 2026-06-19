# 🚀 Intelligent AI Gateway — Smart LLM Router

> **HackIndia Spark 9 | South India Region | Team CodeB**

An intelligent routing layer that automatically selects the most suitable LLM for each task, **minimizing cost and latency** while **preserving output quality**.

---

## 🧠 Problem Statement

Modern AI applications rely on multiple models (GPT-5, Gemini, Claude, DeepSeek, open-source models), but most systems:

| ❌ Problem | 💡 Our Solution |
|---|---|
| Send every request to one expensive model | **Complexity Router** classifies tasks and routes to optimal model |
| Use fixed rules that don't adapt | **Adaptive Router** learns from outcomes via Thompson Sampling |
| Waste money on simple tasks | **Cost-aware routing** sends simple queries to cheaper models |
| Sacrifice quality with cheaper models | **Quality Router** ensures complex tasks get top-tier models |
| No fallback when models fail | **Automatic fallback** with cooldown and retry mechanisms |
| No cost/latency visibility | **Prometheus metrics + spend tracking** for full observability |

### Example

An enterprise copilot receives these requests:

| Task | Without Gateway | With Our Gateway |
|---|---|---|
| "Summarize this email" | GPT-5 ($$$) | GPT-4o-mini (¢) |
| "Generate SQL queries" | GPT-5 ($$$) | GPT-4o ($$) |
| "Write production code" | GPT-5 ($$$) | Claude Sonnet ($$$) — specialized |
| "Analyze this PDF" | GPT-5 ($$$) | Gemini Flash (¢) |
| "Create financial report with reasoning" | GPT-5 ($$$) | o1-preview ($$$) — reasoning tier |

**Result**: ~60-70% cost reduction with equal or better quality.

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    AI Gateway Layer                       │
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │
│  │  Complexity  │  │   Adaptive   │  │    Quality      │ │
│  │   Router     │  │    Router    │  │    Router       │ │
│  │ (Rule-based) │  │  (Bandit ML) │  │ (Tier-aware)    │ │
│  └──────┬──────┘  └──────┬───────┘  └────────┬────────┘ │
│         │                │                    │          │
│         └────────┬───────┴────────────────────┘          │
│                  ▼                                       │
│  ┌──────────────────────────────────────────────┐        │
│  │          Intelligent Router Engine            │        │
│  │  • Fallback chains  • Cooldown management     │        │
│  │  • Rate limiting    • Budget enforcement      │        │
│  │  • Health checks    • Retry policies          │        │
│  └──────────────────────┬───────────────────────┘        │
│                         │                                │
│  ┌──────────────────────┴───────────────────────┐        │
│  │           Observability Layer                 │        │
│  │  • Token usage    • Cost tracking             │        │
│  │  • Latency metrics • Routing explanations     │        │
│  │  • Prometheus     • Spend analytics           │        │
│  └──────────────────────────────────────────────┘        │
└───────────────────────────┬──────────────────────────────┘
                            │
        ┌───────────┬───────┼───────┬────────────┐
        ▼           ▼       ▼       ▼            ▼
   ┌────────┐ ┌────────┐ ┌─────┐ ┌───────┐ ┌─────────┐
   │ GPT-5  │ │ Claude │ │Gemini│ │DeepSeek│ │ Open    │
   │        │ │        │ │      │ │       │ │ Source  │
   └────────┘ └────────┘ └─────┘ └───────┘ └─────────┘
```

---

## 🔑 Key Features

### 1. Task Understanding — Complexity Router
> Zero API calls, sub-millisecond classification

Scores each request across **7 dimensions**:

| Dimension | What it Detects | Weight |
|---|---|---|
| Token Count | Short = simple, long = complex | 10% |
| Code Presence | `function`, `class`, `docker`, etc. | 30% |
| Reasoning Markers | "step by step", "analyze" | 25% |
| Technical Terms | Domain complexity indicators | 25% |
| Simple Indicators | "what is", "define" (negative) | 5% |
| Multi-Step Patterns | "first...then", numbered steps | 3% |
| Question Complexity | Multiple question marks | 2% |

Routes to **4 tiers**:

| Tier | Score | Example Model | Use Case |
|---|---|---|---|
| SIMPLE | < 0.15 | GPT-4o-mini | Greetings, basic Q&A |
| MEDIUM | 0.15–0.35 | GPT-4o | Standard queries |
| COMPLEX | 0.35–0.60 | Claude Sonnet | Technical, multi-part |
| REASONING | > 0.60 | o1-preview | Chain-of-thought analysis |

### 2. Self-Learning — Adaptive Router
> Thompson Sampling bandit that learns which model performs best for each task type

- Classifies requests into 7 task categories: `code_generation`, `writing`, `analytical_reasoning`, `factual_lookup`, `creative`, `extraction`, `general`
- Maintains Beta(α, β) posteriors per `(task_type, model)` pair
- Post-call hooks analyze response quality via regex + tool-call detectors
- Balances **quality × 0.7 + cost × 0.3** by default

### 3. Cost Optimization
- **Lowest-cost routing**: Automatically picks the cheapest model that meets quality requirements
- **Budget limiting**: Per-user, per-team, per-organization budget enforcement
- **Token-per-minute routing**: Distributes load across models by TPM/RPM availability

### 4. Reliability & Fallback
- **Automatic fallback chains**: If model A fails → try model B → try model C
- **Cooldown management**: Temporarily removes failing models from rotation
- **Health state caching**: Monitors model health in real-time
- **Retry policies**: Configurable retry with exponential backoff
- **Context window fallbacks**: Automatically switch to models with larger context

### 5. Observability
- **Prometheus metrics**: Token usage, cost, latency, routing decisions
- **Spend tracking**: Per-key, per-team, per-user cost analytics
- **Custom logging hooks**: Extensible callback system for any observability platform
- **Routing explanations**: Every decision is traceable

---

## 📁 Project Structure

```
├── litellm/
│   ├── router.py                          # Core intelligent router engine
│   ├── router_strategy/
│   │   ├── complexity_router/             # Rule-based task classification
│   │   ├── adaptive_router/               # ML-based Thompson sampling router
│   │   ├── auto_router/                   # Semantic intent routing
│   │   ├── quality_router/                # Quality-tier-aware routing
│   │   ├── lowest_cost.py                 # Cost optimization strategy
│   │   ├── lowest_latency.py              # Latency optimization strategy
│   │   ├── budget_limiter.py              # Budget enforcement
│   │   └── tag_based_routing.py           # Tag-based routing rules
│   ├── router_utils/
│   │   ├── fallback_event_handlers.py     # Automatic model fallback
│   │   ├── cooldown_handlers.py           # Failing model cooldown
│   │   └── health_state_cache.py          # Real-time health monitoring
│   ├── caching/                           # Response caching layer
│   ├── litellm_core_utils/
│   │   ├── token_counter.py               # Token counting & estimation
│   │   ├── get_model_cost_map.py          # Model pricing database
│   │   └── llm_cost_calc/                 # Cost calculation engine
│   ├── types/                             # Data models & type definitions
│   ├── proxy/
│   │   ├── route_llm_request.py           # Request routing logic
│   │   ├── spend_tracking/                # Cost analytics
│   │   └── analytics_endpoints/           # Analytics API
│   ├── integrations/
│   │   ├── prometheus.py                  # Prometheus metrics export
│   │   └── custom_logger.py              # Custom observability hooks
│   ├── cost_calculator.py                 # Unified cost calculation
│   ├── budget_manager.py                  # Budget management
│   └── exceptions.py                      # Error handling
├── config/
│   └── proxy_server_config.yaml           # Sample gateway configuration
├── model_prices_and_context_window.json   # 500+ model pricing database
├── Dockerfile                             # Container deployment
├── docker-compose.yml                     # Multi-service orchestration
├── pyproject.toml                         # Python package config
└── .env.example                           # Environment variables template
```

---

## ⚡ Quick Start

### 1. Configuration

```yaml
# config/smart_router.yaml
model_list:
  # Define your model pool
  - model_name: gpt-4o-mini
    litellm_params:
      model: openai/gpt-4o-mini
      api_key: os.environ/OPENAI_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: openai/gpt-4o
      api_key: os.environ/OPENAI_API_KEY

  - model_name: claude-sonnet
    litellm_params:
      model: anthropic/claude-sonnet-4
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: o1-preview
    litellm_params:
      model: openai/o1-preview
      api_key: os.environ/OPENAI_API_KEY

  # Intelligent routing model
  - model_name: smart-router
    litellm_params:
      model: auto_router/complexity_router
      complexity_router_config:
        tiers:
          SIMPLE: gpt-4o-mini
          MEDIUM: gpt-4o
          COMPLEX: claude-sonnet
          REASONING: o1-preview

litellm_settings:
  success_callback: ["prometheus"]
  num_retries: 3
  context_window_fallbacks:
    - gpt-4o-mini: ["gpt-4o"]

router_settings:
  routing_strategy: usage-based-routing-v2
  enable_pre_call_checks: true
```

### 2. Usage

```python
import litellm

# All requests go through the smart router
# Simple question → routes to gpt-4o-mini (cheap)
response = litellm.completion(
    model="smart-router",
    messages=[{"role": "user", "content": "What is Python?"}]
)

# Complex reasoning → routes to o1-preview (powerful)
response = litellm.completion(
    model="smart-router",
    messages=[{"role": "user", "content": 
        "Think step by step: analyze the performance implications "
        "of implementing a distributed consensus algorithm for "
        "our microservices architecture."}]
)

# Code task → routes to claude-sonnet (specialized)
response = litellm.completion(
    model="smart-router",
    messages=[{"role": "user", "content": 
        "Implement a Redis-backed rate limiter class in Python "
        "with sliding window algorithm and async support."}]
)
```

### 3. Docker Deployment

```bash
docker-compose up -d
```

---

## 📊 Observability Dashboard

The gateway exposes Prometheus metrics including:

- `litellm_llm_api_latency_metric` — per-model latency
- `litellm_requests_metric` — request counts per model
- `litellm_spend_metric` — cost tracking per key/team
- `litellm_tokens_used` — token consumption analytics
- `litellm_deployment_cooled_down` — model health status

---

## 🛡️ Reliability Features

```
Request → Router → Model A (fails/timeout)
                     ↓ automatic fallback
                   Model B (rate limited)
                     ↓ automatic fallback
                   Model C (succeeds) ✅
                     ↓
              Cooldown Model A for 60s
              Log incident for analytics
```

---

## 🔧 Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.9+ |
| Gateway Server | FastAPI (via LiteLLM Proxy) |
| Routing Engine | Custom Router with Strategy Pattern |
| ML Component | Thompson Sampling (Beta-Bernoulli Bandit) |
| Classification | Rule-based weighted scoring |
| Observability | Prometheus + Custom Callbacks |
| Caching | In-memory / Redis / Dual Cache |
| Deployment | Docker + Docker Compose |
| Database | PostgreSQL (optional, for persistence) |

---

## 📄 License

This project uses components from [LiteLLM](https://github.com/BerriAI/litellm) (MIT License). See `LICENSE_LITELLM` for details.

---

## 👥 Team CodeB — South India Region

Built with ❤️ for HackIndia Spark 9
