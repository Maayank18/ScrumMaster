# BharatSprint - Technical Design Document

## 1. System Architecture

### 1.1 High-Level Architecture

```
Developer Input (IDE/Chat/PR/Notes)
          ↓
AI Processing Layer (Semantic Context + LLM)
          ↓
Human-in-Loop Approval (Scrum Master)
          ↓
Execution Layer (Jira/GitHub)
          ↓
Learning & Monitoring (Dashboards + Adaptive Paths)
```

### 1.2 Core Design Principle

**AI drafts, humans approve, systems execute** — governance-first automation that maintains human control while eliminating repetitive coordination work.

---

## 2. Technology Stack

### 2.1 Backend Services

- **Runtime:** Node.js 18+ with TypeScript
- **Framework:** Express.js with Helmet security middleware
- **Database:** MongoDB (document storage) + Redis (caching, job queues)
- **Vector Database:** Weaviate (embeddings and semantic search)
- **Authentication:** JWT with 7-day expiration
- **API Documentation:** Swagger/OpenAPI 3.0

### 2.2 Frontend Application

- **Framework:** Next.js 14 with TypeScript
- **UI Library:** Tailwind CSS + Headless UI components
- **State Management:** Zustand for client state
- **Charts:** Recharts for dashboard visualizations
- **PWA:** Service workers for offline capability

### 2.3 IDE Extension

- **Platform:** VS Code Extension API
- **Language:** TypeScript
- **Communication:** REST API to backend service
- **Features:** Code explanation, test generation, blocker flagging

### 2.4 AI/ML Components

- **LLM Provider:** OpenAI GPT-4 Turbo (primary), Llama 2 (on-prem option)
- **Embeddings:** OpenAI text-embedding-ada-002
- **Vector Search:** Weaviate with HNSW indexing
- **NLP Pipeline:** Custom prompt chains for action extraction

---

## 3. Data Architecture

### 3.1 Core Data Models

#### 3.1.1 Input Sources

```typescript
interface InputSource {
  id: string;
  type: 'meeting_notes' | 'slack_message' | 'pr_comment' | 'ide_selection';
  content: string;
  metadata: {
    author: string;
    timestamp: Date;
    source_url?: string;
    repository?: string;
  };
  processing_status: 'pending' | 'processed' | 'failed';
}
```

#### 3.1.2 Action Drafts

```typescript
interface ActionDraft {
  id: string;
  source_id: string;
  title: string;
  description: string;
  acceptance_criteria: string[];
  story_points: number;
  priority: 'low' | 'medium' | 'high' | 'critical';
  labels: string[];
  suggested_assignee?: string;
  confidence_score: number; // 0-100
  repository_links: RepositoryLink[];
  status: 'draft' | 'approved' | 'rejected' | 'pushed_to_jira';
  created_at: Date;
  reviewed_by?: string;
  jira_issue_key?: string;
}
```

#### 3.1.3 Repository Context

```typescript
interface RepositoryLink {
  type: 'file' | 'pr' | 'commit' | 'issue';
  url: string;
  title: string;
  relevance_score: number;
  excerpt?: string;
}
```

#### 3.1.4 Learning Paths

```typescript
interface LearningPath {
  id: string;
  developer_id: string;
  repository: string;
  duration_weeks: number;
  micro_tasks: MicroTask[];
  progress: number; // 0-100
  created_at: Date;
}

interface MicroTask {
  id: string;
  title: string;
  description: string;
  estimated_minutes: number;
  linked_files: string[];
  linked_jira_issue?: string;
  completed: boolean;
}
```

---

## 4. API Design

### 4.1 Input Processing Endpoints

```
POST /api/v1/inputs
- Accept text input from various sources
- Return: input_id, processing_status

GET /api/v1/inputs/{input_id}/actions
- Get extracted action drafts for an input
- Return: ActionDraft[]
```

### 4.2 Action Management Endpoints

```
GET /api/v1/actions/pending
- Get all pending action drafts for review
- Return: ActionDraft[]

PUT /api/v1/actions/{action_id}
- Update action draft (Scrum Master edits)
- Body: Partial<ActionDraft>

POST /api/v1/actions/{action_id}/approve
- Approve action and push to Jira
- Return: jira_issue_key, jira_url
```

### 4.3 Repository Integration Endpoints

```
POST /api/v1/repositories/{repo_id}/index
- Index repository for semantic search
- Process: Clone → Parse → Embed → Store

GET /api/v1/repositories/{repo_id}/search
- Semantic search across repository
- Query params: query, limit, threshold
```

### 4.4 Learning Path Endpoints

```
POST /api/v1/learning-paths
- Generate learning path for developer
- Body: { developer_id, repository, role, current_tasks }

GET /api/v1/learning-paths/{developer_id}
- Get active learning paths for developer
```

---

## 5. Integration Architecture

### 5.1 Jira Integration

- **Authentication:** OAuth 2.0 (3-legged flow)
- **Permissions:** Read projects, Create issues, Update issues
- **Webhook Setup:** Bidirectional sync for issue status updates
- **Rate Limiting:** Respect Jira API limits (10 requests/second)

### 5.2 GitHub/GitLab Integration

- **Authentication:** Personal Access Tokens or GitHub Apps
- **Permissions:** Read repositories, Read pull requests, Read issues
- **Data Sync:** Periodic sync of repository metadata and recent activity

### 5.3 VS Code Extension Architecture

```
VS Code Extension (TypeScript)
          ↓ (REST API calls)
Backend Service (Node.js)
          ↓ (WebSocket for real-time updates)
Scrum Master Dashboard
```

---

## 6. AI Processing Pipeline

### 6.1 Stage 1: Text Ingestion & Preprocessing

```
Raw Text Input
     ↓
PII Redaction (emails, phone numbers)
     ↓
Language Detection
     ↓
Chunking (semantic boundaries)
     ↓
Metadata Extraction (dates, names, priorities)
```

### 6.2 Stage 2: Context Enrichment

```
Text Chunks
     ↓
Generate Embeddings (OpenAI ada-002)
     ↓
Vector Search (Weaviate)
     ↓
Retrieve Relevant: Files, PRs, Commits, Issues
     ↓
Rank by Relevance Score
```

### 6.3 Stage 3: Action Extraction

```
Enriched Context
     ↓
LLM Prompt Chain:
  1. Summarization Prompt
  2. Action Identification Prompt
  3. Ticket Drafting Prompt
     ↓
Structured Output (JSON)
     ↓
Confidence Scoring
```

### 6.4 Stage 4: Human Review Loop

```
AI Draft
     ↓
Scrum Master Dashboard
     ↓
Edit/Approve/Reject
     ↓
If Approved → Jira API Call
     ↓
Webhook Confirmation
```

---

## 7. Security & Privacy

### 7.1 Authentication & Authorization

- **JWT Tokens:** 7-day expiration, refresh token rotation
- **Role-Based Access:** Admin, Scrum Master, Developer, Viewer
- **API Rate Limiting:** 100 requests/minute per user
- **CORS:** Strict origin validation

### 7.2 Data Protection

- **Encryption:** TLS 1.3 in transit, AES-256 at rest
- **PII Handling:** Automatic redaction before LLM processing
- **Audit Trail:** Immutable logs for all approvals and Jira pushes
- **Data Retention:** Configurable per organization (default: 2 years)

### 7.3 Privacy Controls

- **On-Prem Deployment:** Docker containers for air-gapped environments
- **Local LLM Option:** Llama 2 quantized models
- **Data Residency:** Configurable storage regions
- **GDPR Compliance:** Data export, right to delete, consent management

---

## 8. Performance & Scalability

### 8.1 Performance Targets

- **Action Extraction:** < 10 seconds for 2000-word input
- **Dashboard Load:** < 2 seconds
- **IDE Operations:** < 5 seconds (explain/test generation)

### 8.2 Scalability Design

- **Horizontal Scaling:** Stateless backend services
- **Database Sharding:** MongoDB sharding by organization
- **Caching Strategy:** Redis for frequently accessed data
- **CDN:** Static assets served via CDN
- **Load Balancing:** NGINX with health checks

### 8.3 Monitoring & Observability

- **Metrics:** Prometheus + Grafana
- **Logging:** Structured JSON logs with correlation IDs
- **Tracing:** OpenTelemetry for distributed tracing
- **Alerts:** PagerDuty integration for critical issues

---

## 9. Deployment Architecture

### 9.1 Development Environment

```
Docker Compose:
- MongoDB (single instance)
- Redis (single instance)
- Weaviate (single instance)
- Backend API (hot reload)
- Frontend (Next.js dev server)
```

### 9.2 Production Environment

```
Kubernetes Cluster:
- MongoDB ReplicaSet (3 nodes)
- Redis Cluster (3 nodes)
- Weaviate Cluster (3 nodes)
- Backend API (3 replicas)
- Frontend (2 replicas)
- NGINX Ingress Controller
```

### 9.3 On-Premises Deployment

```
Docker Swarm:
- All services containerized
- Local LLM inference (Ollama)
- Air-gapped operation support
- Backup/restore procedures
```

---

## 10. Bharat-Specific Features

### 10.1 Multilingual Support

- **UI Localization:** i18n with Hindi, Tamil, Telugu, Bengali, Marathi
- **Content Translation:** Azure Translator API for ticket descriptions
- **Code Comments:** Multilingual explanations in IDE extension

### 10.2 Low-Bandwidth Optimization

- **Progressive Web App:** Service workers for offline capability
- **Data Compression:** Gzip compression for API responses
- **Lazy Loading:** Component-based loading for dashboard
- **Sync Optimization:** Delta sync for mobile connections

### 10.3 Compliance & Governance

- **Data Localization:** India-specific deployment regions
- **Audit Requirements:** Detailed logging for compliance
- **Custom Workflows:** Configurable approval processes
- **Integration Flexibility:** Support for local tools and processes

---

## 11. Risk Mitigation

### 11.1 Technical Risks

- **LLM Hallucination:** Human approval required, confidence scores displayed
- **API Rate Limits:** Retry queues, exponential backoff
- **Service Dependencies:** Circuit breakers, graceful degradation

### 11.2 Operational Risks

- **Data Loss:** Automated backups, point-in-time recovery
- **Security Breaches:** Regular security audits, penetration testing
- **Performance Degradation:** Auto-scaling, performance monitoring

---

## 12. Future Enhancements

### 12.1 Phase 2 Features

- **Voice Input:** Speech-to-text for meeting recordings
- **Advanced Analytics:** Predictive sprint planning
- **Mobile Apps:** Native iOS/Android applications
- **Workflow Automation:** Custom Jira workflow triggers

### 12.2 Integration Roadmap

- **Slack/Teams:** Native bot integration
- **Azure DevOps:** Alternative to Jira integration
- **GitLab:** Enhanced GitLab support
- **Confluence:** Documentation linking

---

## 13. Unique Selling Propositions (USPs)

### 13.1 Governed Automation
AI drafts combined with mandatory human approval and complete audit trail ensures trust and compliance.

### 13.2 Repository-Aware Tickets
Semantic linking to files, PRs, and commits makes every Jira issue immediately actionable with full context.

### 13.3 Scrum Master-Centric UX
Single dashboard to approve, assign, and measure — designed for the coordination role, not just developers.

### 13.4 In-IDE to Jira Loop
Flag blockers directly in the editor, which become approved tickets with linked context — zero context loss.

### 13.5 Bharat-First Design
Multilingual support, low-bandwidth modes, and on-prem deployment options address real constraints faced by Indian teams and institutions.
