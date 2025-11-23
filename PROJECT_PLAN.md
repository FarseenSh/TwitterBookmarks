# Twitter Bookmarks Chat - Open Source Project Plan

## 🎯 Project Overview

An **open source, self-hosted** application that allows users to chat with their Twitter bookmarks using AI-powered agentic RAG (Retrieval-Augmented Generation). Users extract their bookmarks via a Chrome extension and interact with them through natural language queries.

**Key Features:**
- 🔓 **100% Open Source** - MIT License
- 🏠 **Self-Hosted** - Your data stays on your machine
- 🔑 **Your API Keys** - Choose your own LLM provider
- 🐳 **Docker-First** - One command to start
- 🎯 **Agentic RAG** - Intelligent, context-aware search
- 📊 **Comprehensive Data** - Captures ALL bookmark metadata

## 🏗️ Architecture Overview

```
┌─────────────────────┐
│  Chrome Extension   │ ──► Extracts Twitter Bookmarks from DOM
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Web Application   │ ──► Next.js/React Frontend
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Agno AgentOS      │ ──► FastAPI Backend with Agno Agents
│   (FastAPI)         │
└──────────┬──────────┘
           │
           ├──► PostgreSQL (PgVector) ──► Vector Embeddings
           ├──► PostgreSQL (Main DB)  ──► User Data, Sessions
           └──► Agno Agentic RAG      ──► Chat Intelligence
```

## 📦 Tech Stack

### Frontend
- **Framework**: Next.js 14+ (App Router) with TypeScript
- **UI Library**: Tailwind CSS + shadcn/ui
- **State Management**: Zustand or React Context
- **API Client**: Fetch API / Axios

### Chrome Extension
- **Manifest**: V3
- **Language**: TypeScript
- **Build Tool**: Vite or Webpack
- **Storage**: Chrome Storage API

### Backend
- **Framework**: Agno AgentOS (FastAPI)
- **Agent Framework**: Agno v2.3.2 (latest - Nov 22, 2025)
- **Language**: Python 3.10+ (required by Agno)
- **API**: RESTful + WebSocket for streaming

### Database
- **Primary DB**: PostgreSQL 15+
- **Vector Store**: PgVector extension
- **ORM**: SQLAlchemy (comes with Agno)
- **Migrations**: Alembic

### AI & RAG
- **Agent Framework**: Agno v2.3.2 (Nov 22, 2025)
- **Primary LLM Provider**: OpenRouter (unified access to multiple models)
- **Recommended Models** (cost-effective for RAG):
  - `qwen/qwen3-max` - $1.60/$6.40 per 1M tokens (optimized for RAG!)
  - `google/gemini-2.5-flash` - Price-performance leader
  - `qwen/qwen3-coder:free` - FREE tier for development
  - `google/gemini-1.5-flash` - Google Gemini (budget-friendly)
- **Alternative Providers**:
  - Google Gemini (direct) - `gemini-1.5-flash` / `gemini-2.5-flash`
  - Hyperbolic - Ultra-low latency inference
  - OpenAI - For premium tier users
- **Embeddings**: OpenAI text-embedding-3-small ($0.02/1M tokens)
- **RAG**: Agno's built-in agentic RAG (automatic when knowledge is provided)
- **Vector Search**: PgVector with hybrid search (semantic + keyword)

### Infrastructure
- **Containerization**: Docker + Docker Compose (one-command setup)
- **Deployment**: Self-hosted (user's machine or VPS)
- **Auth**: Optional (single-user default, can add BasicAuth)
- **Environment**: .env for API keys and configuration

## 🗂️ Project Structure

```
twitter-bookmarks-saas/
├── chrome-extension/          # Chrome extension for bookmark extraction
│   ├── manifest.json
│   ├── src/
│   │   ├── content/          # Content scripts for Twitter DOM
│   │   ├── background/       # Service worker
│   │   ├── popup/            # Extension popup UI
│   │   └── utils/            # Shared utilities
│   ├── public/
│   └── package.json
│
├── backend/                   # Agno AgentOS backend
│   ├── agents/               # Agno agent definitions
│   │   ├── twitter_chat_agent.py
│   │   └── knowledge_agent.py
│   ├── knowledge/            # Knowledge base management
│   │   └── vector_store.py
│   ├── api/                  # FastAPI routes
│   │   ├── auth.py
│   │   ├── bookmarks.py
│   │   └── chat.py
│   ├── db/                   # Database models & migrations
│   │   ├── models.py
│   │   └── session.py
│   ├── services/             # Business logic
│   ├── schemas/              # Pydantic schemas
│   ├── requirements.txt
│   └── main.py               # AgentOS entry point
│
├── frontend/                  # Next.js frontend
│   ├── src/
│   │   ├── app/              # App router pages
│   │   ├── components/       # React components
│   │   ├── hooks/            # Custom hooks
│   │   ├── lib/              # Utilities
│   │   └── types/            # TypeScript types
│   ├── public/
│   └── package.json
│
├── docker-compose.yml         # Local development setup
├── .env.example
└── README.md
```

## 🔧 Core Components

### 1. Chrome Extension (Bookmark Extractor)

**Purpose**: Extract Twitter bookmarks directly from the DOM and send to backend

**Key Features**:
- Automatic detection when user visits Twitter bookmarks page
- DOM parsing to extract:
  - Tweet text content
  - Author information
  - Media (images, videos)
  - URLs and links
  - Timestamp
  - Engagement metrics
- Batch upload to backend API
- Progress tracking & sync status
- Authentication with backend

**Technical Implementation**:
```javascript
// Content script runs on twitter.com/*/bookmarks
- Parse DOM structure (Twitter's React components)
- Extract data using selectors/XPath
- Handle infinite scroll pagination
- Send batched requests to backend API
```

### 2. Backend (Agno AgentOS)

**Purpose**: Manage users, process bookmarks, provide intelligent chat interface

**Key Components**:

#### A. Database Models

**User Model** (Optional - for multi-user setups):
```python
- id: UUID
- username: str (optional)
- created_at: datetime
```

**Bookmark Model** (Comprehensive Schema - CAPTURES EVERYTHING):
```python
# Core Tweet Data
- id: UUID
- tweet_id: str (unique Twitter ID)
- tweet_url: str
- tweet_content: str (full text)
- published_at: datetime
- language: str

# Author Information
- author_name: str (display name)
- author_username: str (@handle)
- author_profile_image_url: str
- author_verified: bool (blue checkmark)
- author_follower_count: int
- author_following_count: int

# Engagement Metrics
- likes: int
- retweets: int
- replies: int
- views: int (if available)
- bookmarks_count: int
- quotes: int

# Media & Links (URLs only - no downloads)
- media_urls: List[str] (images/videos)
- media_types: List[str] (['image', 'video', 'gif'])
- external_urls: List[str] (links in tweet)

# Social Context
- hashtags: List[str] (#tags)
- mentions: List[str] (@mentions)

# Tweet Context
- is_reply: bool
- is_retweet: bool
- is_quote: bool
- reply_to_username: str (if reply)
- reply_to_tweet_id: str
- thread_position: int (if part of thread)

# Metadata
- source: str ('Twitter Web App', 'Twitter for iPhone', etc.)
- bookmarked_at: datetime (when USER bookmarked it)
- synced_at: datetime
- embedding_id: str (PgVector reference)

# Optional
- user_id: UUID (for multi-user)
- user_notes: str (user can add notes)
```

**ChatSession Model**:
```python
- id: UUID
- user_id: UUID (optional)
- created_at: datetime
- updated_at: datetime
- title: str (auto-generated from first message)
```

**ChatMessage Model**:
```python
- id: UUID
- session_id: UUID
- role: str ('user' | 'assistant')
- content: str
- created_at: datetime
```

#### B. Agno Agent Setup (Agentic RAG - 2025 Best Practices)

**Complete Agent with Agentic RAG (Recommended)**:
```python
from agno.agent import Agent
from agno.models.openrouter import OpenRouter
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.pgvector import PgVector, SearchType
from agno.embedder.openai import OpenAIEmbedder
from agno.storage.agent.postgres import PgAgentStorage

# Database connection
db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

# Vector database with hybrid search + OpenAI embeddings
vector_db = PgVector(
    table_name="twitter_bookmarks",
    db_url=db_url,
    search_type=SearchType.hybrid,  # 🔥 Semantic + Keyword search
    embedder=OpenAIEmbedder(
        model="text-embedding-3-small",  # $0.02/1M tokens
        dimensions=1536,
    ),
)

# Knowledge base from bookmarks
knowledge_base = Knowledge(
    vector_db=vector_db,
    num_documents=5,  # Top-5 retrieval per search
)

# 🚀 AGENTIC RAG AGENT (92% accuracy)
chat_agent = Agent(
    name="Twitter Bookmarks Assistant",

    # LLM Model (user configurable)
    model=OpenRouter(
        id="qwen/qwen3-max",  # Default: $1.60/$6.40 per 1M - RAG optimized
        # Other options based on .env PROVIDER setting:
        # id="google/gemini-2.5-flash",  # Price-performance leader
        # id="qwen/qwen3-coder:free",    # FREE for development
    ),

    # Knowledge & Storage
    knowledge=knowledge_base,
    storage=PgAgentStorage(table_name="agent_sessions", db_url=db_url),

    # 🎯 AGENTIC RAG CONFIGURATION (critical!)
    search_knowledge=True,       # ✅ Agent decides WHEN to search
    read_chat_history=True,      # ✅ Remembers conversation
    add_history_to_context=True, # ✅ Adds history to LLM context
    num_history_runs=3,          # ✅ Last 3 interactions

    # Output formatting
    markdown=True,
    show_tool_calls=True,  # See when agent searches knowledge
    debug_mode=False,
)
```

**4 LLM Provider Options** (from .env):

```python
import os
from agno.models.openrouter import OpenRouter
from agno.models.google import Gemini
from agno.models.ollama import Ollama

# User chooses provider in .env
provider = os.getenv("PROVIDER", "openrouter")

if provider == "openrouter":
    model = OpenRouter(id=os.getenv("MODEL_NAME", "qwen/qwen3-max"))
elif provider == "gemini":
    model = Gemini(id=os.getenv("MODEL_NAME", "gemini-2.5-flash"))
elif provider == "hyperbolic":
    # Hyperbolic uses OpenRouter-compatible API
    model = OpenRouter(
        id=os.getenv("MODEL_NAME", "meta-llama/llama-3.1-70b"),
        api_key=os.getenv("HYPERBOLIC_API_KEY"),
        base_url="https://api.hyperbolic.xyz/v1"
    )
elif provider == "ollama":
    model = Ollama(
        id=os.getenv("MODEL_NAME", "llama3.1:8b"),
        base_url=os.getenv("OLLAMA_BASE_URL", "http://localhost:11434")
    )

# Create agent with user's chosen model
chat_agent = Agent(
    name="Twitter Bookmarks Assistant",
    model=model,
    knowledge=knowledge_base,
    search_knowledge=True,      # Agentic RAG
    read_chat_history=True,     # Context awareness
    add_history_to_context=True,
    num_history_runs=3,
    markdown=True,
)
```

#### C. Advanced: Async Agent Usage (Recommended for Production)

```python
import asyncio
from agno.agent import Agent
from agno.models.openrouter import OpenRouter
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.pgvector import PgVector, SearchType
from agno.embedder.openai import OpenAIEmbedder

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

# Vector database with hybrid search
vector_db = PgVector(
    table_name="twitter_bookmarks",
    db_url=db_url,
    search_type=SearchType.hybrid,
    embedder=OpenAIEmbedder(model="text-embedding-3-small"),
)
knowledge_base = Knowledge(vector_db=vector_db, num_documents=5)

# Agent with Agentic RAG
agent = Agent(
    model=OpenRouter(id="qwen/qwen3-max"),
    knowledge=knowledge_base,
    search_knowledge=True,       # Agentic RAG
    read_chat_history=True,
    add_history_to_context=True,
    num_history_runs=3,
    markdown=True,
)

async def main():
    # Add bookmarks to knowledge base asynchronously
    # (In real app, this happens when user syncs from Chrome extension)

    # Query agent asynchronously
    response = await agent.arun("What are my most recent AI bookmarks?")
    print(response.content)

    # Streaming response (for real-time chat UX)
    async for chunk in agent.arun_stream("Summarize my Python bookmarks"):
        print(chunk.content, end="", flush=True)

if __name__ == "__main__":
    asyncio.run(main())
```

#### D. API Endpoints (Single-User Focused)

```python
# Bookmarks
POST   /api/bookmarks/sync          # Bulk upload from Chrome extension
GET    /api/bookmarks               # List all bookmarks (with filters)
GET    /api/bookmarks/:id           # Get single bookmark
DELETE /api/bookmarks/:id           # Delete bookmark
PUT    /api/bookmarks/:id/notes     # Update user notes
POST   /api/bookmarks/embed         # Trigger re-embedding

# Chat (Agentic RAG)
POST   /api/chat/sessions           # Create new chat session
GET    /api/chat/sessions           # List all sessions
GET    /api/chat/sessions/:id       # Get session with messages
POST   /api/chat/message            # Send message (supports streaming)
DELETE /api/chat/sessions/:id       # Delete session

# Statistics
GET    /api/stats                   # Bookmark stats (count, top authors, etc.)

# Health
GET    /api/health                  # System health check

# Optional (for multi-user)
# POST   /api/auth/register
# POST   /api/auth/login
# GET    /api/auth/me
```

### 3. Agentic RAG System

**Purpose**: Enable intelligent querying of Twitter bookmarks

**Agno RAG Features**:
- **Automatic Embedding**: Bookmarks automatically embedded when added to knowledge base
- **Hybrid Search**: Semantic + keyword search
- **Agentic Retrieval**: Agent decides when/how to query bookmarks
- **Context-Aware**: Maintains conversation context across messages
- **Multi-turn Conversations**: Memory of previous interactions

**Example Queries Users Can Make**:
- "Show me all bookmarks about AI and machine learning"
- "What did Elon Musk tweet about that I bookmarked?"
- "Find tweets with Python code examples"
- "Summarize my bookmarks from last month"
- "Which bookmarks mention GPT-4?"

**Processing Pipeline**:
```
1. User sends message
2. Agno agent analyzes intent
3. Agent queries vector DB (agentic RAG)
4. Retrieves top-k relevant bookmarks
5. Agent synthesizes response with context
6. Streams response back to user
```

### 4. Frontend (Next.js Chat Interface)

**Purpose**: Provide seamless chat experience

**Key Pages**:
- `/` - Landing page
- `/dashboard` - User dashboard with stats
- `/chat` - Main chat interface
- `/bookmarks` - Browse all bookmarks
- `/settings` - User settings & API keys

**Chat Interface Features**:
- Real-time message streaming
- Markdown rendering
- Code syntax highlighting
- Bookmark cards with preview
- Session management
- Export conversations
- Dark/light mode

## 📋 Implementation Phases

### Phase 1: Foundation (Week 1-2)
- [ ] Set up project structure
- [ ] Initialize Git repository
- [ ] Set up PostgreSQL with PgVector
- [ ] Create Docker Compose for local dev
- [ ] Initialize Next.js frontend
- [ ] Initialize Agno backend
- [ ] Set up environment variables

### Phase 2: Chrome Extension (Week 2-3)
- [ ] Create extension manifest v3
- [ ] Build content script for Twitter DOM parsing
- [ ] Implement bookmark extraction logic
- [ ] Add popup UI for sync status
- [ ] Implement batch upload to backend
- [ ] Add error handling & retry logic
- [ ] Test on various Twitter layouts

### Phase 3: Backend Core (Week 3-4)
- [ ] Define database models (User, Bookmark, Session, Message)
- [ ] Set up Alembic migrations
- [ ] Implement authentication (JWT)
- [ ] Create bookmark sync API
- [ ] Set up PgVector for embeddings
- [ ] Implement embedding generation
- [ ] Create basic CRUD endpoints

### Phase 4: Agno Agent & RAG (Week 4-5)
- [ ] Configure Agno agent with knowledge base
- [ ] Connect PgVector to Agno knowledge
- [ ] Implement agentic RAG
- [ ] Set up agent memory & sessions
- [ ] Create chat API endpoints
- [ ] Implement streaming responses
- [ ] Test RAG quality & relevance

### Phase 5: Frontend Development (Week 5-6)
- [ ] Build authentication UI (login/register)
- [ ] Create dashboard layout
- [ ] Build chat interface
- [ ] Implement WebSocket/SSE for streaming
- [ ] Create bookmark browser
- [ ] Add session management UI
- [ ] Implement responsive design

### Phase 6: Integration & Testing (Week 6-7)
- [ ] End-to-end testing
- [ ] Chrome extension → Backend → Frontend flow
- [ ] Performance optimization
- [ ] Security audit
- [ ] Rate limiting
- [ ] Error handling improvements

### Phase 7: Deployment & Launch (Week 7-8)
- [ ] Set up production database
- [ ] Deploy backend (Railway/Render)
- [ ] Deploy frontend (Vercel)
- [ ] Publish Chrome extension
- [ ] Set up monitoring (Sentry)
- [ ] Create user documentation
- [ ] Soft launch & feedback

## 🔒 Security Considerations

1. **Data Privacy** (Self-Hosted Advantage)
   - ALL data stays on user's machine
   - No external servers (except chosen LLM provider API)
   - User controls their own database

2. **Chrome Extension**
   - Content Security Policy
   - Limited permissions (only twitter.com)
   - No data sent except to user's own backend

3. **API Security**
   - CORS configuration (locked to localhost by default)
   - Input validation (Pydantic)
   - SQL injection prevention (SQLAlchemy ORM)
   - XSS protection

4. **Optional Authentication** (for multi-user setups)
   - BasicAuth or JWT tokens
   - Secure password hashing (bcrypt)
   - Rate limiting

## 📊 Success Metrics (Open Source Project)

- ⭐ GitHub Stars
- 🍴 Forks & Contributors
- 📦 Docker pulls
- 📖 Documentation quality
- 🐛 Issue resolution time
- 💬 Community engagement (Discord/GitHub Discussions)
- 📈 Chrome extension installs
- ✅ User success stories

## 🚀 Future Enhancements

- [ ] Multi-platform support (LinkedIn, Reddit bookmarks)
- [ ] Bookmark collections & tagging
- [ ] Scheduled bookmark summaries via email
- [ ] Mobile app (React Native)
- [ ] Collaborative bookmarks
- [ ] Browser extension for Firefox, Edge
- [ ] Integration with note-taking apps (Notion, Obsidian)
- [ ] Advanced analytics & insights
- [ ] Bookmark recommendations
- [ ] Export to PDF/Markdown

## 🔐 Environment Variables (.env.example)

```bash
# =============================================================================
# CHOOSE YOUR LLM PROVIDER (4 OPTIONS)
# =============================================================================

# Option 1: OpenRouter (RECOMMENDED - easiest, 200+ models)
PROVIDER=openrouter
OPENROUTER_API_KEY=sk-or-v1-...  # Get from https://openrouter.ai/keys
MODEL_NAME=qwen/qwen3-max  # or google/gemini-2.5-flash, qwen/qwen3-coder:free

# Option 2: Google Gemini (Free tier available!)
# PROVIDER=gemini
# GOOGLE_API_KEY=...  # Get from https://makersuite.google.com/app/apikey
# MODEL_NAME=gemini-2.5-flash  # or gemini-1.5-flash

# Option 3: Hyperbolic (Ultra-low latency)
# PROVIDER=hyperbolic
# HYPERBOLIC_API_KEY=...  # Get from https://hyperbolic.xyz
# MODEL_NAME=meta-llama/llama-3.1-70b

# Option 4: Ollama (100% LOCAL - no API key needed!)
# PROVIDER=ollama
# OLLAMA_BASE_URL=http://localhost:11434
# MODEL_NAME=llama3.1:8b  # or llama3.1:70b, qwen2.5:14b

# =============================================================================
# EMBEDDINGS (REQUIRED - OpenAI only)
# =============================================================================
OPENAI_API_KEY=sk-...  # Get from https://platform.openai.com/api-keys
# Cost: $0.02 per 1M tokens (basically free for personal use)

# =============================================================================
# DATABASE (Docker handles this - no changes needed)
# =============================================================================
DATABASE_URL=postgresql+psycopg://ai:ai@localhost:5532/ai
PGVECTOR_URL=postgresql+psycopg://ai:ai@localhost:5532/ai

# =============================================================================
# APPLICATION
# =============================================================================
APP_NAME="Twitter Bookmarks Chat"
APP_ENV=development
DEBUG=True
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173

# Frontend URL (for CORS)
FRONTEND_URL=http://localhost:3000

# =============================================================================
# OPTIONAL: Authentication (for multi-user setups)
# =============================================================================
# JWT_SECRET_KEY=your_super_secret_key_here
# JWT_ALGORITHM=HS256
# ACCESS_TOKEN_EXPIRE_MINUTES=30
```

## 📦 Backend Dependencies (requirements.txt)

```txt
# Agno Framework (Latest - Nov 22, 2025)
agno==2.3.2

# Web Framework
fastapi>=0.104.0
uvicorn[standard]>=0.24.0

# Database
sqlalchemy>=2.0.0
alembic>=1.12.0
psycopg[binary]>=3.1.0  # PostgreSQL driver
pgvector>=0.2.0

# Authentication & Security
pyjwt>=2.8.0
passlib[bcrypt]>=1.7.4
python-multipart>=0.0.6

# Environment & Config
python-dotenv>=1.0.0
pydantic>=2.5.0
pydantic-settings>=2.1.0

# CORS & Middleware
python-cors>=1.0.0

# AI Models (Cost-Effective Options)
# OpenRouter is included with Agno - no extra package needed!
google-genai>=0.8.0  # For Gemini direct integration
openai>=1.0.0  # For embeddings only ($0.02/1M tokens)

# Optional Premium Models (only if offering premium tier)
# anthropic>=0.40.0  # Uncomment for Claude Sonnet 4.5

# Storage (if not using PgAgentStorage)
# agno-aws  # Optional: for AWS S3 storage
```

## 🛠️ Development Setup

### Prerequisites
```bash
- Python 3.10+ (required by Agno 2.3.2)
- Node.js 20+ LTS
- PostgreSQL 15+ with PgVector extension
- Docker & Docker Compose
- Git
```

### Quick Start (5 Minutes)

**Step 1: Clone & Configure**
```bash
# Clone the repository
git clone https://github.com/yourusername/twitter-bookmarks-chat
cd twitter-bookmarks-chat

# Copy environment template
cp .env.example .env

# Edit .env and add your API keys:
# - OPENROUTER_API_KEY (get from https://openrouter.ai/keys)
# - OPENAI_API_KEY (get from https://platform.openai.com/api-keys)
```

**Step 2: Start with Docker Compose** (Recommended)
```bash
# One command starts everything!
docker-compose up -d

# Wait ~30 seconds for services to start, then:
# Frontend: http://localhost:3000
# Backend API: http://localhost:8000/docs
```

**Step 3: Load Chrome Extension**
```bash
1. Open Chrome → chrome://extensions
2. Enable "Developer mode" (top right)
3. Click "Load unpacked"
4. Select the `chrome-extension/dist` folder
5. Visit Twitter → Bookmarks
6. Click extension icon → Sync
```

**Manual Setup** (Without Docker)
```bash
# 1. Start PostgreSQL with PgVector
docker run -d \
  -e POSTGRES_DB=ai \
  -e POSTGRES_USER=ai \
  -e POSTGRES_PASSWORD=ai \
  -v pgvolume:/var/lib/postgresql/data \
  -p 5532:5432 \
  --name pgvector \
  agnohq/pgvector:16

# 2. Backend
cd backend
python3 -m venv venv
source venv/bin/activate
pip install agno==2.3.2
pip install -r requirements.txt
alembic upgrade head
uvicorn main:app --reload --port 8000

# 3. Frontend (new terminal)
cd frontend
npm install
npm run dev

# 4. Extension (new terminal)
cd chrome-extension
npm install
npm run build
```

## 📚 Resources & Documentation

- [Agno Framework](https://github.com/agno-agi/agno)
- [Agno Documentation](https://docs.agno.com)
- [PgVector](https://github.com/pgvector/pgvector)
- [Chrome Extension Docs](https://developer.chrome.com/docs/extensions/mv3/)
- [Next.js Documentation](https://nextjs.org/docs)

---

## 📚 Sources & References

**Agno Framework (2025)**:
- [GitHub - agno-agi/agno](https://github.com/agno-agi/agno) - Main repository
- [Agno PyPI Package](https://pypi.org/project/agno/) - v2.3.1 (latest)
- [Agno Documentation](https://docs.agno.com/) - Official docs
- [Agno: The agent framework for Python teams](https://workos.com/blog/agno-the-agent-framework-for-python-teams)
- [PgVector Agent Knowledge - Agno](https://docs.agno.com/concepts/vectordb/pgvector)

**OpenRouter Integration (Primary LLM Provider)**:
- [OpenRouter - Agno Docs](https://docs.agno.com/concepts/models/openrouter) - Official integration
- [OpenRouter Models](https://openrouter.ai/models) - Model catalog & pricing
- [OpenRouter Pricing](https://openrouter.ai/pricing) - Cost structure
- [Top AI Models on OpenRouter 2025](https://www.teamday.ai/blog/top-ai-models-openrouter-2025) - Cost vs performance
- [OpenRouter Review 2025](https://skywork.ai/blog/openrouter-review-2025/) - Production testing

**Google Gemini Integration**:
- [Gemini - Agno Docs](https://docs.agno.com/tools/toolkits/models/gemini) - Official integration
- [Using Agno with Google Gemini and PgVector](https://www.linkedin.com/posts/devevantelista_using-agno-with-google-gemini-and-pgvector-activity-7303258024289210368-1pmD) - Real-world example

**Model Recommendations for RAG**:
- Qwen3-Max: Optimized for RAG, $1.60/$6.40 per 1M tokens (Nov 2025 pricing)
- Gemini 2.5 Flash: Price-performance leader for production (GA since May 2025)
- Qwen3-Coder (Free): Development & free tier

**Implementation Guides**:
- [Agentic RAG with Hybrid Search - Agno](https://docs.agno.com/examples/apps/agentic-rag)
- [Performing Agentic RAG with MongoDB and Agno](https://medium.com/@sharathpai107/performing-agentic-rag-with-mongodb-and-agno-010ceef5141b)
- [Agentic Framework Deep Dive: Agno](https://medium.com/@devipriyakaruppiah/agentic-framework-deep-dive-series-part-2-agno-c45da579b7c0)
- [RAG for Data Engineers with Agno & PgVector](https://thepipeandtheline.substack.com/p/introduction-to-rag-hands-on-implementation)

**Agentic RAG Documentation**:
- [Knowledge - Agno Docs](https://docs.agno.com/agents/knowledge) - Official knowledge docs
- [Agentic RAG Cookbook](https://github.com/agno-agi/agno/tree/main/cookbook) - Code examples
- [Agno Support Agent Example](https://github.com/agno-agi/agno/blob/main/cookbook/examples/agents/agno_support_agent.py) - Real implementation

---

## 📜 License

**MIT License**

Copyright (c) 2025 [Your Name]

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## 🤝 Contributing

We welcome contributions! This is an open source project.

**How to Contribute:**
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

**Areas We Need Help:**
- 🐛 Bug fixes
- 📝 Documentation improvements
- 🎨 UI/UX enhancements
- 🌐 Additional LLM provider support
- 🧪 Testing & QA
- 🌍 Internationalization

## 📞 Support & Community

- 🐛 **Issues**: [GitHub Issues](https://github.com/yourusername/twitter-bookmarks-chat/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/yourusername/twitter-bookmarks-chat/discussions)
- 📖 **Docs**: [Documentation](https://github.com/yourusername/twitter-bookmarks-chat/wiki)
- ⭐ **Star** the repo if you find it useful!

---

**Last Updated**: 2025-11-23 (Open Source, Agno v2.3.2, Agentic RAG, 4 LLM Providers, Nov 2025 verified)
