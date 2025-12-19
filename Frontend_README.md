# RaptorRelay Control Plane

The RaptorRelay Control Plane is a purpose-built visualization and interaction layer for the RaptorRelay Automation platform.

It provides a clear, real-time view of:
- Security incidents
- SOAR workflow execution
- AI-assisted decisions
- Automated and human-approved responses

The Control Plane is designed as a **thin orchestration UI**, not a replacement for StackStorm, OpenSearch Dashboards, or OpenProject.


## Goals

- Provide a single, polished demo surface for RaptorRelay
- Make AI-assisted decisions explainable and auditable
- Show end-to-end incident flow in real time
- Enable fast demos using synthetic incidents
- Run fully containerized with minimal setup


## Non-Goals

- Replacing existing SOC tooling
- Implementing complex user management
- Serving as a long-term analytics warehouse
- Becoming a general-purpose SIEM UI


## Architecture Overview

````
┌─────────────────────┐
│  Control Plane UI   │  (React / Next.js)
└─────────┬───────────┘
│ REST / WebSocket
┌─────────▼───────────┐
│ Control Plane API   │  (FastAPI / Node)
└─────────┬───────────┘
│
┌────────▼─────────┐   ┌──────────────┐   ┌───────────────┐
│  StackStorm API  │   │ OpenSearch   │   │ OpenProject   │
└──────────────────┘   └──────────────┘   └───────────────┘
│
┌────────▼─────────┐
│ AI Agent Outputs │ (JSON artifacts)
└──────────────────┘

````


## Core Features

- Live incident timeline
- Active incident dashboard
- AI decision transparency panel
- Synthetic incident injector
- Exportable incident reports


## Running the Demo

```bash
docker-compose -f docker-compose.demo.yaml up
```

This will start:
* Control Plane UI
* Control Plane API
* StackStorm
* OpenSearch + Dashboards
* Ollama
* Demo data injector


## Directory Structure

```
demo/
├── dashboard/
│   ├── frontend/
│   └── backend/
├── injector/
├── reports/
└── docker-compose.demo.yaml
```


## Audience

* Security leaders
* Platform engineers
* SOC architects
* Open-source contributors

This Control Plane exists to **tell the RaptorRelay story clearly**.

---

# Product Requirement Documents (PRDs)

## PRD-001: Control Plane Frontend

### Objective
Provide a visually polished, intuitive UI that communicates system state, decisions, and outcomes in under 3 minutes.

### Target Users
- SOC leads
- Security architects
- Executive stakeholders (read-only)

### Key Views

#### A. Active Incidents
- List of current incidents
- Severity, status, AI risk score
- Linked ticket ID
- Automation state

#### B. Incident Timeline (Primary View)
Each incident displays:
- Detection event
- Enrichment output
- MITRE mapping
- AI decision
- Response action
- Human approval (if any)

Each node must show:
- Timestamp
- Source system
- Summary
- Raw JSON (expandable)

#### C. AI Decision Panel
- Agent name
- Input summary
- Output decision
- Confidence score
- Explanation snippet

#### D. Demo Controls
- “Run Demo Incident” button
- Progress animation
- Reset environment button

### UX Constraints
- No raw logs by default
- No configuration editing
- Read-only with limited action triggers


## PRD-002: Control Plane Backend API

### Objective
Aggregate data from multiple backend systems into a single, stable API for the frontend.

### Responsibilities
- Normalize incident data
- Poll or subscribe to backend systems
- Expose read-only REST endpoints
- Support WebSocket updates

### Required Integrations

| System        | Method            |
|--------------|-------------------|
| StackStorm   | REST API polling  |
| OpenSearch   | Query + scroll    |
| OpenProject  | REST API          |
| AI Agents    | File or object store JSON |

### Core Endpoints

````

GET /incidents
GET /incidents/{id}
GET /incidents/{id}/timeline
GET /incidents/{id}/ai-decisions
POST /demo/inject
GET /reports/{id}

````

### Data Model (Canonical Incident)

```json
{
  "incident_id": "string",
  "severity": "low|medium|high|critical",
  "status": "open|in_progress|resolved",
  "mitre_techniques": [],
  "ai_risk_score": 0.0,
  "timeline": [],
  "linked_ticket": "string"
}
```


## PRD-003: Demo Incident Injector

### Objective

Provide deterministic, repeatable demo scenarios.

### Behavior

* Injects synthetic event into OpenSearch
* Triggers StackStorm sensor
* Tracks incident ID
* Feeds known data to AI agents

### Scenarios

* Malware detection
* Privilege escalation
* Lateral movement

### Constraints

* Must not affect real data
* Must be resettable


## PRD-004: Exportable Incident Reports

### Objective

Generate executive-ready artifacts.

### Formats

* PDF
* JSON
* Markdown

### Contents

* Incident summary
* Timeline
* AI decisions
* MITRE mapping
* Actions taken


# Explicit Integration Instructions

## For Google Antigravity AI

### Step 1: Backend Alignment

* Treat existing StackStorm packs as **sources of truth**
* Do not modify SOAR logic for UI needs
* Emit AI agent outputs as structured JSON artifacts:

  ```
  /var/raptorrelay/ai_outputs/{incident_id}.json
  ```


### Step 2: Control Plane API

* Create a new service:

  ```
  demo/dashboard/backend
  ```
* Implement adapters:

  * `StackStormAdapter`
  * `OpenSearchAdapter`
  * `OpenProjectAdapter`
  * `AIDecisionAdapter`
* Normalize all data into the canonical incident model


### Step 3: Frontend Integration

* Frontend **never** calls backend systems directly
* All calls go through Control Plane API
* Use polling first; WebSockets optional but encouraged
* Render timeline strictly from API-provided ordering


### Step 4: Demo Mode Contract

* Demo injector returns:

  ```json
  { "incident_id": "demo-123" }
  ```
* Frontend auto-navigates to that incident
* Timeline animates as backend updates arrive


### Step 5: Docker Contract

* Each component runs as a single container
* No shared databases
* All configuration via environment variables
* Demo compose file is authoritative