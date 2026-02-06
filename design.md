# BharatSprint - System Design Document

## 1. Design Overview

BharatSprint is a **microservices-based, AI-augmented coordination platform** built on the MERN stack with LLM integration and vector search capabilities.

**Design Principles:**
1. **Human-in-loop governance:** AI assists, humans decide
2. **Jira as source of truth:** We reference, not mirror
3. **Modularity:** Swappable LLM providers, storage backends
4. **Privacy-first:** On-prem deployment option, minimal data retention
5. **Bharat-optimized:** Low bandwidth, multilingual, offline-capable

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Layer                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Next.js    │  │  VS Code Ext │  │  Mobile PWA  │      │
│  │  Dashboard   │  │ (TypeScript) │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓ HTTPS/REST
┌─────────────────────────────────────────────────────────────┐
│                   API Gateway Layer                          │
│              (Express.js + TypeScript)                       │
│          ┌──────────────────────────────────┐               │
│          │  Auth, Rate Limiting, CORS       │               │
│          └──────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                  Application Services Layer                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Ingestion    │  │  AI Service  │  │ Jira Service │      │
│  │  Service     │  │   (LLM)      │  │   (OAuth)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Learning    │  │ Code Search  │  │IDE Assistant │      │
│  │   Service    │  │  (Vector)    │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                     Data Layer                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   MongoDB    │  │  Weaviate    │  │    Redis     │      │
│  │  (Primary)   │  │  (Vectors)   │  │  (Cache/Q)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                 External Integrations                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Jira Cloud  │  │    GitHub    │  │   OpenAI     │      │
│  │   (OAuth)    │  │   (REST)     │  │   (API)      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

---

### 2.2 Component Diagram

```
┌────────────────────────────────────────────────────────────────┐
│                     BharatSprint Platform                       │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                  Frontend Components                      │ │
│  │  • Scrum Master Dashboard (action review, metrics)       │ │
│  │  • Developer Portal (learning paths, profile)            │ │
│  │  • Admin Panel (project settings, integrations)          │ │
│  └──────────────────────────────────────────────────────────┘ │
│                              ↓                                  │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                  Backend Services                         │ │
│  │                                                           │ │
│  │  ┌────────────────┐  ┌────────────────┐                 │ │
│  │  │ Ingestion API  │  │  AI Processor  │                 │ │
│  │  │ - Upload notes │  │  - Extract     │                 │ │
│  │  │ - Slack hook   │  │  - Enrich      │                 │ │
│  │  │ - PR webhook   │  │  - Draft       │                 │ │
│  │  └────────────────┘  └────────────────┘                 │ │
│  │                                                           │ │
│  │  ┌────────────────┐  ┌────────────────┐                 │ │
│  │  │ Jira Connector │  │ Code Analyzer  │                 │ │
│  │  │ - OAuth flow   │  │  - Repo scan   │                 │ │
│  │  │ - Issue CRUD   │  │  - Embeddings  │                 │ │
│  │  │ - Webhook sync │  │  - Search      │                 │ │
│  │  └────────────────┘  └────────────────┘                 │ │
│  │                                                           │ │
│  │  ┌────────────────┐  ┌────────────────┐                 │ │
│  │  │ Learning Gen   │  │ IDE Service    │                 │ │
│  │  │ - Path create  │  │  - Explain     │                 │ │
│  │  │ - Task assign  │  │  - Test gen    │                 │ │
│  │  │ - Progress     │  │  - Flag block  │                 │ │
│  │  └────────────────┘  └────────────────┘                 │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## 3. Data Models

### 3.1 MongoDB Schema Design

#### User Collection
```javascript
{
  _id: ObjectId,
  organizationId: ObjectId,  // ref: Organization
  email: String (unique),
  password: String (hashed),
  name: String,
  role: Enum['scrum_master', 'developer', 'student', 'manager'],
  jiraAccountId: String (optional),
  createdAt: Date
}
```

#### Project Collection
```javascript
{
  _id: ObjectId,
  organizationId: ObjectId,
  name: String,
  repoUrl: String,
  repoProvider: Enum['github', 'gitlab'],
  jiraProjectKey: String (optional),
  settings: {
    language: String (default: 'en'),
    allowExternalLLM: Boolean (default: true),
    onPremInference: Boolean (default: false)
  },
  createdAt: Date
}
```

#### ActionDraft Collection
```javascript
{
  _id: ObjectId,
  inputSourceId: ObjectId,  // ref: InputSource
  projectId: ObjectId,      // ref: Project
  status: Enum['draft', 'pending_review', 'approved', 'rejected', 'pushed_to_jira'],
  
  // AI extracted
  title: String,
  description: Text,
  acceptanceCriteria: Text (Gherkin format),
  suggestedAssigneeId: ObjectId (ref: User, optional),
  suggestedStoryPoints: Number [1,2,3,5,8],
  labels: Array<String>,
  confidenceScore: Number [0-100],
  sourceExcerpt: Text,
  
  // Human review
  reviewedById: ObjectId (ref: User, optional),
  reviewedAt: Date (optional),
  createdAt: Date,
  updatedAt: Date
}
```

#### JiraIssue Collection
```javascript
{
  _id: ObjectId,
  actionDraftId: ObjectId (ref: ActionDraft),
  jiraIssueKey: String,    // e.g., "PROJ-123"
  jiraIssueId: String,
  jiraUrl: String,
  status: String,
  lastSyncedAt: Date,
  createdAt: Date
}
```

**Indexes:**
```javascript
// Performance-critical indexes
User: { email: 1 } (unique)
ActionDraft: { projectId: 1, status: 1 }
JiraIssue: { jiraIssueKey: 1 } (unique)
CodeContext: { projectId: 1, filePath: 1 }
```

---

### 3.2 Weaviate Schema (Vector Database)

```javascript
{
  class: "CodeChunk",
  properties: [
    { name: "projectId", dataType: ["string"] },
    { name: "filePath", dataType: ["string"] },
    { name: "content", dataType: ["text"] },
    { name: "language", dataType: ["string"] },
    { name: "chunkType", dataType: ["string"] },  // function, class, comment
    { name: "metadata", dataType: ["object"] }
  ],
  vectorizer: "text2vec-openai"
}
```

**Usage:** Semantic search for "find code related to payment processing"

---

## 4. API Design

### 4.1 REST API Endpoints

#### Authentication
```
POST   /api/auth/register          # Create account
POST   /api/auth/login             # Get JWT token
GET    /api/auth/me                # Get current user
```

#### Ingestion
```
POST   /api/ingest                 # Submit text input
Body: {
  projectId: string,
  sourceType: 'meeting_notes' | 'slack' | 'pr_comment' | 'ide',
  content: string,
  metadata: object
}
Response: { inputSourceId: string, status: 'processing' }
```

#### Actions
```
GET    /api/projects/:projectId/actions              # List action drafts
Query: ?status=pending_review
Response: { actions: ActionDraft[] }

PATCH  /api/actions/:actionId                        # Edit draft (Scrum Master)
Body: { title?, description?, acceptanceCriteria?, ... }
Response: { action: ActionDraft }

POST   /api/actions/:actionId/push-to-jira           # Create Jira issue
Body: { sprintId?: number }
Response: { jiraIssueKey: string }

POST   /api/actions/:actionId/rejecum Master, Developer, Viewer
- **API Rate Limiting:** 100 requests/minute per user
- **CORS:** Strict origin validation

### Data Protection
- **Encryption:** TLS 1.3 in transit, AES-256 at rest
- **PII Handling:** Automatic redaction before LLM processing
- **Audit Trail:** Immutable logs for all approvals and Jira pushes
- **Data Retention:** Configurable per organization (default: 2 years)

### Privacy Controls
- **On-Prem Deployment:** Docker containers for air-gapped environments
- **Local LLM Option:** Llama 2 quantized models
- **Data Residency:** Configurable storage regions
- **GDPR Compliance:** Data export, right to delete, consent management

## Performance & Scalability

### Performance Targets
- **Action Extraction:** < 10 seconds for 2000-word input
- **Dashboard Load:** < 2 seconds
- **IDE Operations:** < 5 seconds (explain/test generation)

### Scalability Design
- **Horizontal Scaling:** Stateless backend services
- **Database Sharding:** MongoDB sharding by organization
- **Caching Strategy:** Redis for frequently accessed data
- **CDN:** Static assets served via CDN
- **Load Balancing:** NGINX with health checks

### Monitoring & Observability
- **Metrics:** Prometheus + Grafana
- **Logging:** Structured JSON logs with correlation IDs
- **Tracing:** OpenTelemetry for distributed tracing
- **Alerts:** PagerDuty integration for critical issues

## Deployment Architecture

### Development Environment
```
Docker Compose:
- MongoDB (single instance)
- Redis (single instance)  
- Weaviate (single instance)
- Backend API (hot reload)
- Frontend (Next.js dev server)
```

### Production Environment
```
Kubernetes Cluster:
- MongoDB ReplicaSet (3 nodes)
- Redis Cluster (3 nodes)
- Weaviate Cluster (3 nodes)
- Backend API (3 replicas)
- Frontend (2 replicas)
- NGINX Ingress Controller
```

### On-Premises Deployment
```
Docker Swarm:
- All services containerized
- Local LLM inference (Ollama)
- Air-gapped operation support
- Backup/restore procedures
```

## Bharat-Specific Features

### Multilingual Support
- **UI Localization:** i18n with Hindi, Tamil, Telugu, Bengali, Marathi
- **Content Translation:** Azure Translator API for ticket descriptions
- **Code Comments:** Multilingual explanations in IDE extension

### Low-Bandwidth Optimization
- **Progressive Web App:** Service workers for offline capability
- **Data Compression:** Gzip compression for API responses
- **Lazy Loading:** Component-based loading for dashboard
- **Sync Optimization:** Delta sync for mobile connections

### Compliance & Governance
- **Data Localization:** India-specific deployment regions
- **Audit Requirements:** Detailed logging for compliance
- **Custom Workflows:** Configurable approval processes
- **Integration Flexibility:** Support for local tools and processes

## Risk Mitigation

### Technical Risks
- **LLM Hallucination:** Human approval required, confidence scores displayed
- **API Rate Limits:** Retry queues, exponential backoff
- **Service Dependencies:** Circuit breakers, graceful degradation

### Operational Risks
- **Data Loss:** Automated backups, point-in-time recovery
- **Security Breaches:** Regular security audits, penetration testing
- **Performance Degradation:** Auto-scaling, performance monitoring

## Future Enhancements

### Phase 2 Features
- **Voice Input:** Speech-to-text for meeting recordings
- **Advanced Analytics:** Predictive sprint planning
- **Mobile Apps:** Native iOS/Android applications
- **Workflow Automation:** Custom Jira workflow triggers

### Integration Roadmap
- **Slack/Teams:** Native bot integration
- **Azure DevOps:** Alternative to Jira integration
- **GitLab:** Enhanced GitLab support
- **Confluence:** Documentation linking