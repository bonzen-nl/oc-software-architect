# Software Architect — OpenClaw Skill

**Intelligente orchestrator voor model-routing, cost-tracking en project management.**

Analyzeert task-complexiteit, routeert naar optimaal AI-model, tracked kosten, beheert projecten. Volledig autonoom met Telegram-rapportage.

---

## 🎯 Wat doet Software Architect?

Software Architect is de **overkoepelende orchestrator-skill** die:

- **Task Complexity Analysis** — Analyzeert code/tekst/beschrijvingen → Level 1/2/3
- **Smart Model Routing** — Kiest optimaal model (Ollama → Claude → OpenAI → Gemini)
- **Cost Estimation** — Voordat execution: hoeveel tokens/geld?
- **Budget Control** — Monthly limits per provider, auto-failover
- **Project Management** — Per-project cost tracking + history
- **Telegram Alerts** — Model switches, budget warnings, completions
- **Token Database** — Alles gelogd in SQLite voor later analyseren

### 🔄 Routing Decision Tree

```
Task Input (code/description/file)
    ↓
ComplexityAnalyzer
    ├─ Level 1: Simple (typos, comments, rename)
    │  └─→ Ollama (Mistral) — FREE
    │
    ├─ Level 2: Medium (features, debugging, API)
    │  └─→ Claude 3.5 Sonnet — €0.003-0.015/K tokens
    │
    └─ Level 3: Complex (architecture, security, performance)
       ├─ Budget check: OpenAI available?
       │  ├─ Yes → GPT-4o (best quality)
       │  └─ No → Gemini 1.5 (cheaper backup)
       └─ On Claude error → Fallback to OpenAI
```

---

## 📦 Afhankelijkheden

### Systeemvereisten
- **Python:** 3.8+
- **SQLite3:** (included with Python)
- **Ollama:** Optional (Level 1 tasks, free local processing)

### Python Dependencies

```
anthropic>=0.7.0              # Claude API (primary)
openai>=1.0.0                 # OpenAI API (fallback)
google-generativeai>=0.3.0    # Gemini API (backup)
requests>=2.28.0              # HTTP calls
python-dotenv>=0.20.0         # .env loading
```

### API Keys Required
- **Anthropic (Claude):** ANTHROPIC_API_KEY
- **OpenAI (optional):** OPENAI_API_KEY
- **Gemini (optional):** GEMINI_API_KEY
- **Telegram (optional):** TELEGRAM_BOT_TOKEN, TELEGRAM_CHAT_ID

---

## ⚡ Quickstart

### 1. Installatie

```bash
# Clone repository
git clone https://github.com/bonzen-nl/oc-software-architect
cd oc-software-architect

# Virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Dependencies
pip install -r requirements.txt
```

### 2. Configuratie

```bash
# Copy template
cp .env.example .env

# Fill in (minimum: Anthropic API key):
# ANTHROPIC_API_KEY=sk-ant-xxx
# OPENAI_API_KEY=sk-xxx (optional)
# GEMINI_API_KEY=xxx (optional)
# TELEGRAM_BOT_TOKEN=xxx
# TELEGRAM_CHAT_ID=your_chat_id
```

### 3. Initialize Database

```bash
# Create token_usage.db (one-time)
python3 scripts/init_database.py

# Verify
sqlite3 token_usage.db "SELECT * FROM model_calls LIMIT 1;"
```

### 4. Test Routing

```bash
# Estimate cost (dry-run, no execution)
./software-architect route "Refactor authentication module" \
  --type task \
  --project my-app

# Expected output:
# {
#   "complexity_level": 2,
#   "selected_provider": "anthropic",
#   "estimated_tokens": 1250,
#   "estimated_cost_eur": 0.025,
#   "within_budget": true
# }
```

---

## 🚀 Gebruik

### Dry-Run Mode (Estimate Only)

```bash
# Analyze without execution
./software-architect route \
  "Implement OAuth2 authentication" \
  --type task \
  --project backend-api \
  --dry-run

# Output: Complexity level + estimated cost + model choice
```

### Execute & Track

```bash
# Analyze + execute + log
./software-architect route \
  "Build REST API with FastAPI" \
  --type task \
  --project backend-api \
  --execute

# Output: Actual cost + tokens used + logged to token_usage.db
```

### Project Status

```bash
# View all projects
./software-architect status

# View specific project
./software-architect status backend-api

# Output:
# {
#   "project_id": "backend-api",
#   "name": "Backend API Project",
#   "total_cost_eur": 45.23,
#   "models_used": ["anthropic", "openai"],
#   "total_tokens": 125000
# }
```

### Create Project

```bash
# Initialize project
./software-architect project create \
  --project-id my-app \
  --name "My Application" \
  --budget-eur 100

# Output: Project created, ready for tracking
```

---

## 🏗️ Projectstructuur

```
oc-software-architect/
├── SKILL.md                          # Skill documentatie
├── README.md                         # Dit bestand
├── software-architect               # CLI wrapper
├── requirements.txt                 # Python dependencies
├── .env.example                     # Configuration template
├── .gitignore                       # Git security
├── LICENSE                          # MIT
├── token_usage.db                   # SQLite database (generated)
├── config/
│   └── software_architect.json      # Routing rules + budgets
├── lib/
│   ├── expert.py                    # Core orchestrator
│   ├── complexity_analyzer.py       # Task analysis
│   ├── model_router.py              # Provider selection
│   ├── cost_tracker.py              # SQLite logging
│   └── notifier.py                  # Telegram alerts
├── scripts/
│   ├── init_database.py             # Setup token_usage.db
│   └── project_status.py            # Project management
└── .venv/                           # Virtual environment
```

---

## 📊 Complexity Levels

### Level 1 — Simple (Ollama)
**Keywords:** typo, comment, docstring, rename, trivial

- **Examples:**
  - Fix typos in code
  - Add missing docstrings
  - Rename variables for clarity
  - Simple refactoring

- **Provider:** Mistral Small 24B (Ollama, local)
- **Cost:** FREE ✅
- **Context Window:** 4K tokens

### Level 2 — Medium (Claude)
**Keywords:** feature, integration, refactor, debug, API

- **Examples:**
  - Implement new features
  - API integrations
  - Performance debugging
  - Code optimization

- **Provider:** Claude 3.5 Sonnet
- **Cost:** €0.003-0.015 per 1K tokens
- **Context Window:** 32K tokens

### Level 3 — Complex (Claude → Fallback)
**Keywords:** architecture, security, performance, migration, scalability

- **Examples:**
  - System redesign
  - Security audits
  - Major performance optimization
  - Database migration planning

- **Primary:** Claude 3.5 Sonnet
- **Budget Check:** OpenAI monthly budget available?
  - Yes → GPT-4o (best quality)
  - No → Gemini 1.5 Pro (cheaper)
- **Cost:** €0.02-0.05 per task
- **Context Window:** 100K+ tokens

---

## 💰 Cost Control

### Budget Management

```json
{
  "providers": {
    "anthropic": {"monthly_budget": "unlimited"},
    "openai": {"monthly_budget_eur": 50.0},
    "gemini": {"monthly_budget_eur": 20.0}
  },
  "alerts": {
    "alert_at_percent": 75,
    "max_single_task_cost": 25.0
  }
}
```

### Auto-Failover Logic

1. Estimate cost
2. If cost > max → Alert Bob for approval
3. If within budget → Execute
4. On provider error → Try fallback
5. Update token_usage.db

### Example Flow

```
Task: "Redesign authentication system"
    ↓
Complexity: Level 3 (Complex)
    ↓
Primary: Claude (estimated €0.05)
    ↓
OpenAI budget: €37.50 / €50 (75%)
    ↓
Alert: "Budget threshold reached, continue with Claude?"
    ↓
User approves
    ↓
Execute Claude
    ↓
Log: €0.048 spent, total now €37.55
```

---

## 📚 Database Schema

### model_calls Table

```sql
CREATE TABLE model_calls (
  id INTEGER PRIMARY KEY,
  timestamp TEXT,
  model TEXT,
  provider TEXT,
  input_tokens INTEGER,
  output_tokens INTEGER,
  cost_eur REAL,
  project_id TEXT,
  task_type TEXT,
  complexity_level INTEGER,
  status TEXT
);
```

### projects Table

```sql
CREATE TABLE projects (
  project_id TEXT PRIMARY KEY,
  name TEXT,
  created_at TEXT,
  total_cost_eur REAL,
  total_input_tokens INTEGER,
  total_output_tokens INTEGER,
  last_updated TEXT
);
```

---

## 🔐 Veiligheid

### API Keys
- Stored in `.env` only (600 permissions)
- Never logged to console
- Rotated periodically recommended
- Use provider-specific limits if available

### Database
- SQLite file contains cost data (no secrets)
- Safe to version control (no API keys)
- Can be shared for cost audits

---

## 🧪 Testing

### Test Routing

```bash
# Level 1 task
./software-architect route "Fix typo in README" --type task

# Level 2 task
./software-architect route "Implement user authentication" --type task

# Level 3 task
./software-architect route "Redesign database schema for performance" --type task
```

### Cost Estimation

```bash
# Multiple estimates
python3 scripts/test_cost_estimation.py

# Output: Accuracy of token predictions
```

---

## 🐛 Troubleshooting

### "ANTHROPIC_API_KEY not found"
```bash
# Check .env
cat .env | grep ANTHROPIC_API_KEY

# Regenerate key: https://console.anthropic.com/
```

### "Budget exceeded, operation blocked"
```bash
# Check project costs
./software-architect status project-name

# Increase budget in config/software_architect.json
```

### "Fallback model failed, no more options"
```bash
# All providers exhausted (rare)
# Check API status: anthropic.com, openai.com, google.com
# Retry: add --retry flag
```

---

## 🔗 Sub-Projecten & Integraties

Software Architect is de **orchestrator voor het hele ecosystem**:

### Master Hub
- **[oc-overzicht](https://github.com/bonzen-nl/oc-overzicht)** — Central index

### Gerelateerde Skills (Orchestrated)
- **[oc-github-manager](https://github.com/bonzen-nl/oc-github-manager)** — Architect calls for repo setup
- **[oc-openclaw-expert](https://github.com/bonzen-nl/oc-openclaw-expert)** — Architect consults for context
- **[oc-server-status](https://github.com/bonzen-nl/oc-server-status)** — Architect reads system metrics
- **[oc-ram-guardian](https://github.com/bonzen-nl/oc-ram-guardian)** — Architect aware of memory constraints

### Integration Flow

```
User Request
    ↓
Software-Architect (analyze)
    ↓
├─→ Consult openclaw-expert (context)
├─→ Check server-status (resources)
├─→ Verify ram-guardian (memory OK?)
├─→ Execute task (Claude/Ollama/OpenAI)
├─→ Call github-manager (store results)
└─→ Log to token_usage.db
    ↓
Telegram Alert (Bob notified)
```

---

## 📈 Performance Metrics

- **Complexity analysis:** <1 sec
- **Cost estimation:** <500ms
- **Token prediction:** 85% accuracy
- **Database logging:** <100ms
- **Telegram notifications:** <2 sec (network dependent)

---

## 📝 Licentie

MIT © 2026 Bonzen

---

## 📬 Ondersteuning

- **Issues:** [oc-software-architect/issues](https://github.com/bonzen-nl/oc-software-architect/issues)
- **Integration Help:** Zie skill-specifieke repos

---

**Onderdeel van:** [OpenClaw Skills Suite](https://github.com/bonzen-nl/oc-overzicht)

**Status:** 🟢 PRODUCTION — Orchestrator for entire ecosystem
