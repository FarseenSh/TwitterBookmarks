# Twitter Bookmarks Chat SaaS - Project Plan

## 🎯 Project Overview

A SaaS application that allows users to chat with their Twitter bookmarks using AI-powered agentic RAG (Retrieval-Augmented Generation). Users can extract their Twitter bookmarks via a Chrome extension and interact with them through natural language queries.

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
- **Agent Framework**: Agno v2.3.1 (latest - Nov 2025)
- **Language**: Python 3.10+ (required by Agno)
- **API**: RESTful + WebSocket for streaming

### Database
- **Primary DB**: PostgreSQL 15+
- **Vector Store**: PgVector extension
- **ORM**: SQLAlchemy (comes with Agno)
- **Migrations**: Alembic

### AI & RAG
- **Agent Framework**: Agno v2.3.1
- **Primary LLM Provider**: OpenRouter (unified access to multiple models)
- **Recommended Models** (cost-effective for RAG):
  - `qwen/qwen3-max` - $1.2/$6 per 1M tokens (optimized for RAG!)
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
- **Containerization**: Docker + Docker Compose
- **Auth**: NextAuth.js / Clerk / Supabase Auth
- **Deployment**: Vercel (Frontend) + Railway/Render (Backend)
- **Environment**: .env management

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
```python
# User model
- id, email, created_at, subscription_tier

# Bookmark model
- id, user_id, tweet_id, content, author, media_urls,
  created_at, synced_at, embedding_id

# ChatSession model
- id, user_id, created_at, updated_at

# ChatMessage model
- id, session_id, role, content, created_at
```

#### B. Agno Agent Setup (Updated 2025 Syntax with Cost-Effective Models)

**Option 1: OpenRouter (Recommended - Best Cost/Performance)**
```python
from agno.agent import Agent
from agno.models.openrouter import OpenRouter
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.pgvector import PgVector, SearchType
from agno.storage.agent.postgres import PgAgentStorage

# Database connection
db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

# Vector database for bookmarks with hybrid search
vector_db = PgVector(
    table_name="twitter_bookmarks",
    db_url=db_url,
    search_type=SearchType.hybrid,  # Combines semantic + keyword search
)

# Knowledge base from bookmarks
knowledge_base = Knowledge(
    vector_db=vector_db,
    num_documents=5,  # Top-k retrieval
)

# Chat agent with RAG using OpenRouter
chat_agent = Agent(
    name="Twitter Bookmarks Assistant",
    model=OpenRouter(
        id="qwen/qwen3-max",  # $1.2/$6 per 1M - optimized for RAG!
        # Alternative options:
        # id="google/gemini-2.5-flash",  # Price-performance leader
        # id="qwen/qwen3-coder:free",    # FREE for development
    ),
    knowledge=knowledge_base,
    storage=PgAgentStorage(table_name="agent_sessions", db_url=db_url),
    read_chat_history=True,  # Maintains conversation context
    markdown=True,
    show_tool_calls=True,
    debug_mode=False,
)
```

**Option 2: Google Gemini Direct (Also Cost-Effective)**
```python
from agno.agent import Agent
from agno.models.google import Gemini
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.pgvector import PgVector, SearchType

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

vector_db = PgVector(
    table_name="twitter_bookmarks",
    db_url=db_url,
    search_type=SearchType.hybrid,
)

knowledge_base = Knowledge(vector_db=vector_db, num_documents=5)

# Chat agent with Gemini
chat_agent = Agent(
    name="Twitter Bookmarks Assistant",
    model=Gemini(id="gemini-2.5-flash"),  # Fast & cheap
    knowledge=knowledge_base,
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

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

# Create vector database and knowledge base
vector_db = PgVector(
    table_name="twitter_bookmarks",
    db_url=db_url,
    search_type=SearchType.hybrid,
)
knowledge_base = Knowledge(vector_db=vector_db)

# Create agent with OpenRouter
agent = Agent(
    model=OpenRouter(id="qwen/qwen3-max"),  # Cost-effective RAG model
    knowledge=knowledge_base,
    markdown=True,
)

async def main():
    # Load knowledge asynchronously (faster for large datasets)
    await knowledge_base.add_content_async(
        url="https://example.com/bookmarks.pdf"
    )

    # Query agent asynchronously
    response = await agent.arun("What are my most recent AI bookmarks?")
    print(response.content)

    # Streaming response (for real-time chat UX)
    async for chunk in agent.arun_stream("Summarize these bookmarks"):
        print(chunk.content, end="", flush=True)

if __name__ == "__main__":
    asyncio.run(main())
```

#### D. API Endpoints
```python
# Authentication
POST   /api/auth/register
POST   /api/auth/login
GET    /api/auth/me

# Bookmarks
POST   /api/bookmarks/sync          # Bulk upload from extension
GET    /api/bookmarks               # List user bookmarks
DELETE /api/bookmarks/:id           # Delete bookmark
POST   /api/bookmarks/embed         # Trigger embedding generation

# Chat
POST   /api/chat/sessions           # Create new chat session
GET    /api/chat/sessions           # List user sessions
GET    /api/chat/sessions/:id       # Get session with messages
POST   /api/chat/message            # Send message (streaming)
DELETE /api/chat/sessions/:id       # Delete session
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

1. **Authentication**
   - JWT tokens with refresh mechanism
   - Secure password hashing (bcrypt)
   - Rate limiting on auth endpoints

2. **Data Privacy**
   - User bookmarks are private by default
   - Encrypted data at rest
   - HTTPS only communication

3. **Chrome Extension**
   - Content Security Policy
   - Limited permissions
   - Secure token storage

4. **API Security**
   - CORS configuration
   - Input validation (Pydantic)
   - SQL injection prevention (SQLAlchemy ORM)
   - XSS protection

## 💰 Monetization Strategy & Cost Analysis

### Cost Breakdown (Using OpenRouter)

**Per User Monthly Costs (Estimated)**:
- **Free Tier** (50 messages/month):
  - Model: `qwen/qwen3-coder:free` - $0/month
  - Embeddings: ~1,000 bookmarks @ $0.02/1M tokens ≈ $0.001
  - Database: Minimal (shared PostgreSQL)
  - **Total: ~$0.001/user** 🎯 Highly profitable!

- **Pro Tier** (500 messages/month):
  - Model: `qwen/qwen3-max` @ $1.2/$6 per 1M tokens
  - Average usage: ~200K tokens input, 100K tokens output
  - Cost: (200K × $1.2 + 100K × $6) / 1M = $0.24 + $0.60 = **$0.84/month**
  - Embeddings: ~$0.01
  - **Total: ~$0.85/user** → **89% margin at $9/month!** 🚀

- **Team Tier** (Unlimited):
  - Model: `google/gemini-2.5-flash` (faster, scalable)
  - Average 2,000 messages/user/month
  - Cost: ~$2-3/user/month
  - **Total: ~$3/user** → **90% margin at $29/month!** 💰

### Pricing Tiers

1. **Free Tier** - $0/month
   - Up to 100 bookmarks
   - 50 chat messages/month
   - Basic RAG with free model
   - Community support

2. **Pro Tier** - $9/month
   - Unlimited bookmarks
   - 500 chat messages/month
   - Advanced RAG with Qwen3-Max
   - Email support
   - Export features

3. **Team Tier** - $29/month
   - Everything in Pro
   - Unlimited messages
   - Faster model (Gemini 2.5 Flash)
   - Shared bookmarks
   - Team collaboration
   - API access
   - Priority support

4. **Enterprise** - Custom pricing
   - Claude Sonnet 4.5 (premium model)
   - Custom integrations
   - Dedicated support
   - SLA guarantees

## 📊 Success Metrics

- User registrations
- Chrome extension installs
- Bookmarks synced
- Chat messages sent
- User retention rate
- Query satisfaction (thumbs up/down)
- Conversion rate (free → paid)

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

## 🔐 Environment Variables (.env)

```bash
# Database
DATABASE_URL=postgresql+psycopg://ai:ai@localhost:5532/ai
PGVECTOR_URL=postgresql+psycopg://ai:ai@localhost:5532/ai

# AI Models (Primary - OpenRouter)
OPENROUTER_API_KEY=your_openrouter_api_key_here  # Get from openrouter.ai
OPENROUTER_APP_NAME=TwitterBookmarksChat  # Optional: for tracking

# AI Models (Alternative Providers)
GOOGLE_API_KEY=your_google_api_key_here  # For Gemini direct
HYPERBOLIC_API_KEY=your_hyperbolic_key_here  # Optional: Hyperbolic AI
OPENAI_API_KEY=your_openai_api_key_here  # Optional: for embeddings

# Model Selection (configurable per tier)
DEFAULT_MODEL=qwen/qwen3-max  # Free tier: qwen3-coder:free
PRO_MODEL=google/gemini-2.5-flash  # Pro tier
TEAM_MODEL=anthropic/claude-sonnet-4-5  # Premium option

# JWT Authentication
JWT_SECRET_KEY=your_super_secret_key_here_change_in_production
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Application
APP_NAME="Twitter Bookmarks Chat"
APP_ENV=development
DEBUG=True
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173

# Frontend URL (for CORS)
FRONTEND_URL=http://localhost:3000

# AgentOS Config
AGNO_API_KEY=  # Optional: for AgentOS cloud features
AGNO_WORKSPACE_ID=  # Optional: for multi-tenant setups
```

## 📦 Backend Dependencies (requirements.txt)

```txt
# Agno Framework (Latest - Nov 2025)
agno==2.3.1

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
- Python 3.10+ (required by Agno 2.3.1)
- Node.js 20+ LTS
- PostgreSQL 15+ with PgVector extension
- Docker & Docker Compose
- Git
```

### Quick Start
```bash
# Clone repository
git clone <repo-url>
cd twitter-bookmarks-saas

# Start PostgreSQL with PgVector using official Agno image
docker run -d \
  -e POSTGRES_DB=ai \
  -e POSTGRES_USER=ai \
  -e POSTGRES_PASSWORD=ai \
  -e PGDATA=/var/lib/postgresql/data/pgdata \
  -v pgvolume:/var/lib/postgresql/data \
  -p 5532:5432 \
  --name pgvector \
  agnohq/pgvector:16

# Backend setup
cd backend
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install Agno and dependencies (latest version 2.3.1)
pip install agno
pip install -r requirements.txt

# Run migrations
alembic upgrade head

# Start AgentOS backend
uvicorn main:app --reload --port 8000

# Frontend setup (in new terminal)
cd frontend
npm install
npm run dev

# Chrome extension (in new terminal)
cd chrome-extension
npm install
npm run build
# Load unpacked extension in Chrome at chrome://extensions
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
- Qwen3-Max: Optimized for RAG, $1.2/$6 per 1M tokens
- Gemini 2.5 Flash: Price-performance leader for production
- Qwen3-Coder (Free): Development & free tier

**Implementation Guides**:
- [Agentic RAG with Hybrid Search - Agno](https://docs.agno.com/examples/apps/agentic-rag)
- [Performing Agentic RAG with MongoDB and Agno](https://medium.com/@sharathpai107/performing-agentic-rag-with-mongodb-and-agno-010ceef5141b)
- [Agentic Framework Deep Dive: Agno](https://medium.com/@devipriyakaruppiah/agentic-framework-deep-dive-series-part-2-agno-c45da579b7c0)
- [RAG for Data Engineers with Agno & PgVector](https://thepipeandtheline.substack.com/p/introduction-to-rag-hands-on-implementation)

**Last Updated**: 2025-11-23 (Agno v2.3.1, OpenRouter primary, cost-optimized)
