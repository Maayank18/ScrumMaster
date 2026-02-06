# BharatSprint - Requirements Document

## Problem Statement
Development teams lose 4-10 hours per sprint manually converting conversations, meeting notes, and PR discussions into structured Jira tickets. Junior developers struggle with generic learning resources disconnected from their actual codebase, leading to 8-12 week onboarding times.

## Solution Overview
BharatSprint is an AI-powered Scrum Master that converts developer input directly into governed, context-rich Jira tickets while accelerating onboarding through code-tied learning paths.

**Core Value Proposition:**
- Reduce sprint coordination overhead by 60%
- Accelerate onboarding (time-to-first-merged-PR) by 40%
- Maintain human control with AI assistance, not replacement

## User Personas

### Primary: Scrum Master (Priya)
- Manages 8-person dev team, runs 2-week sprints
- **Pain:** Spends 6+ hours/sprint converting meeting notes to tickets
- **Goal:** Reduce admin work, improve action-item completion

### Secondary: Junior Developer (Rahul)
- 6 months experience, learning team's codebase
- **Pain:** Generic tutorials don't match team's code patterns
- **Goal:** Learn faster, contribute meaningful PRs within first month

### Tertiary: Senior Developer (Anjali)
- Tech lead, mentors juniors, reviews PRs
- **Pain:** Repetitive questions, unclear ticket requirements
- **Goal:** Spend less time clarifying, more time coding

## Functional Requirements

### FR-001: Text Input Ingestion
**Description:** Accept text inputs from multiple sources
- **Inputs:** Meeting notes, Slack messages, PR comments, IDE selections
- **Output:** Processed input with metadata (author, timestamp, source)

### FR-002: AI Action Extraction
**Description:** Extract actionable items from text using LLM
- **Processing:** 
  - Semantic chunking and NER for owners, dates, priorities
  - LLM prompt chain: summarization → action extraction → ticket drafting
- **Output:** Action drafts with title, description, acceptance criteria, story points, labels, confidence score

### FR-003: Repository Context Linking
**Description:** Automatically link actions to relevant code
- **Processing:** Semantic search across repo files, PRs, commits, issues
- **Output:** Links to files, PRs, commits in ticket description

### FR-004: Human-in-Loop Review
**Description:** Scrum Master reviews and edits AI drafts
- **Features:** List view of pending drafts, inline editing, evidence display, bulk actions
- **Validation:** Cannot push to Jira without explicit approval

### FR-005: Jira Integration
**Description:** Create Jira issues from approved actions
- **Authentication:** OAuth 2.0
- **Capabilities:** Create issues, assign to sprints, sync status via webhooks

### FR-006: In-IDE Assistant
**Description:** VS Code extension for contextual help
- **Features:** Explain code, generate unit tests, find related PRs, flag blockers
- **Technical:** TypeScript extension, REST API to backend

### FR-007: Code-Tied Learning Paths
**Description:** Generate personalized learning from repo analysis
- **Processing:** Analyze code patterns, detect skill gaps, generate curriculum
- **Output:** Micro-tasks (30-60 min) linked to actual repo files

### FR-008: Team Health Dashboard
**Description:** Metrics and insights for Scrum Master
- **Metrics:** Action completion rate, blocker aging, learning adoption, sentiment trends

## User Stories

### Scrum Master Stories
- **US-001:** As a Scrum Master, I want to upload meeting notes and get AI-drafted Jira tickets with acceptance criteria, so I can save time on ticket creation
- **US-002:** As a Scrum Master, I want to review and edit AI drafts before they're pushed to Jira, so I maintain control over sprint quality
- **US-003:** As a Scrum Master, I want to see which developers are blocked in real-time, so I can intervene early

### Junior Developer Stories
- **US-004:** As a junior developer, I want a learning path based on our actual repo, so I can learn patterns we actually use
- **US-005:** As a junior developer, I want to flag blockers from my IDE without context-switching, so I can get unblocked faster
- **US-006:** As a junior developer, I want code explanations in Hindi when I'm stuck, so I can understand complex logic in my native language

### Senior Developer Stories
- **US-007:** As a senior developer, I want tickets to have clear acceptance criteria and linked code context, so I don't need to ask for clarification
- **US-008:** As a senior developer, I want to see which juniors are learning which parts of the codebase, so I can mentor effectively

## Acceptance Criteria (MVP Demo)

For hackathon demo, system must:
- ✅ Accept text input (meeting notes)
- ✅ Extract 3+ action items with acceptance criteria and story points
- ✅ Display in Scrum Master dashboard for review
- ✅ Push 1+ item to demo Jira project
- ✅ Show VS Code extension (explain code, flag blocker)
- ✅ Generate 1-week learning path from sample repo

## Success Metrics

| Metric | Baseline | Target (3 months) |
|--------|----------|-------------------|
| Time spent on ticket creation | 6 hrs/sprint | 2.4 hrs/sprint (60% reduction) |
| Onboarding time (time-to-first-PR) | 8 weeks | 4.8 weeks (40% reduction) |
| Action-item completion rate | 60% | 85% |
| Developer NPS | N/A | > 40 |

## Out of Scope (MVP)
- Voice/audio input
- Multi-repository support
- Advanced workflow automation
- Mobile native apps
- Real-time collaboration

## Constraints
- **Technical:** Requires OpenAI API or self-hosted LLM
- **Business:** MVP Budget: $500/month
- **Timeline:** 3 days for hackathon demo
- **Regulatory:** Indian data residency requirements (addressed via on-prem option)