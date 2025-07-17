# OVR-114 Runbook Discovery & Incident Resolution Plan

## 1 | Goal  
Deliver an AI-assisted system that discovers Confluence runbooks, embeds them for semantic search, and orchestrates an incident workflow through Jira and Slack—first as a **prototype**, then as an **enterprise-grade** solution.

# 2 | Prototype (User Stories & Acceptance)

| # | Title | As a / I want / So that | Acceptance Criteria |
|---|-------|-------------------------|-------------------|
| **P-1** | Basic Workflow Orchestration | **As a** MC-DBA engineer**I want** LangGraph workflow that finds runbooks automatically**So that** I get suggestions instantly without manual search | -  Jira ticket fetched via MCP-  Top-3 runbooks returned-  End-to-end processing under 30s-  Error handling for MCP failures |
| **P-2** | Slack Team Notifications | **As a** MC-DBA team member**I want** automated Slack alerts with runbook recommendations**So that** team has rapid incident awareness | -  Message posted to #mc-dba-jira-notifications-  Rich formatting with incident details-  Thread creation for follow-up-  Notification delivery under 5s |
| **P-3** | Runbook Gap Detection | **As a** MC-DBA engineer**I want** system to detect missing runbooks**So that** I can identify knowledge gaps and create procedures | -  Gap flagged when relevance -  Gap analysis suggests runbook categories-  Incident stored for knowledge improvement-  Team alerted when gaps detected |
## 3 | Prototype (Architecture)

| Layer | Strategy Interface | Prototype Implementation | Swap-out Options |
|-------|-------------------|--------------------------|------------------|
| Runbook Discovery | `RunbookDiscoveryStrategy` | Confluence CQL search (title/text "runbook") | Git, SharePoint |
| Vector Storage | `VectorStorageStrategy` | ChromaDB local collection | Weaviate, Milvus |
| Doc Storage (Hierarchy) | `DocumentStorageStrategy` | Markdown files mirroring Confluence tree | Object store, Git |
| Orchestration | LangGraph via GraphMCP SDK | "Nodes: fetch → search → notify → comment" | Any workflow engine |
| Persistence | — *(prototype skips RDBMS)* | Files + Chroma metadata only | PostgreSQL layer in prod |

### 3.1 Prototype Workflow (flowchart)

```mermaid
flowchart TD
    A["Jira Incident Assigned"] --> B["Fetch Incident(Jira MCP)"]
    B --> C["Semantic Search(Runbook MCP)"]
    C --> D{"Runbooks Found?"}
    D -->|Yes| E["Format Results"]
    D -->|No| F["Gap Analysis"]
    E --> G["Add Jira Comment"]
    F --> G
    G --> H["Slack Notify Team"]
```

### 3.2 Prototype interfaces

### Core MCP Server Architecture

```mermaid
classDiagram
    class DbIncidentResolverMCPServer {
        +string name
        +string version
        +GraphMCPOrchestrator orchestrator
        +RunbookDiscoveryStrategy runbook_strategy
        +VectorStorageStrategy vector_strategy
        +DataPersistenceStrategy persistence_strategy
        +configure_strategies()
        +handle_tool_call(tool_name, arguments)
        +process_incident(jira_key)
        +search_runbooks(query, client_filter)
        +handle_runbook_gap(incident_id, gap_analysis)
    }

    class GraphMCPOrchestrator {
        +dict clients
        +StateGraph workflow_graph
        +execute_workflow(workflow_name, initial_state)
        +_build_langgraph_workflow()
        +_fetch_incident_node(state)
        +_search_runbooks_node(state)
        +_notify_team_node(state)
        +_format_results_node(state)
    }

    class WorkflowState {
        +string jira_key
        +dict incident_data
        +list runbooks
        +bool gap_detected
        +dict gap_analysis
        +string slack_thread
        +dict client_config
    }

    DbIncidentResolverMCPServer --> GraphMCPOrchestrator
    GraphMCPOrchestrator --> WorkflowState
```

### Strategy Pattern Interfaces

```mermaid
classDiagram
    class RunbookDiscoveryStrategy {
        >
        +discover_runbooks(spaces)
        +get_runbook_content(runbook_id)
        +validate_runbook_content(page)
        +extract_runbook_metadata(page)
    }

    class VectorStorageStrategy {
        >
        +store_runbook(runbook_id, content, metadata)
        +search_similar(query, filters)
        +delete_runbook(runbook_id)
        +get_embedding_metadata()
    }

    class DataPersistenceStrategy {
        >
        +store_incident(incident_data)
        +track_runbook_usage(incident_id, runbook_id, score)
        +get_incident_history(incident_id)
        +update_effectiveness_metrics(runbook_id, success)
    }

    class NotificationStrategy {
        >
        +send_team_notification(incident_data, runbooks)
        +send_approval_request(incident_id, proposed_solution)
        +send_escalation_alert(incident_id, engineer)
    }
```

## 4 | Post-Prototype (User Stories)

| # | Title | Key Extras (beyond prototype) |
|---|-------|------------------------------|
| Prod-1 | Full 14-Step Incident Workflow | Diagnostics, approval gates, remediation execution, client closure |
| Prod-2 | Real-Time Runbook Sync | Confluence webhooks, automatic vector refresh |
| Prod-3 | Effectiveness Analytics | Track runbook success, SLA metrics, trend dashboards |
| Prod-4 | High Availability | Weaviate cluster + PostgreSQL replication, health monitoring |

### 4.1 Target Production Workflow (simplified view)

```mermaid
flowchart TD
    A1["1-2 Arrival & Slack Alert"]
    A2["3-4 AI Runbook Search & Link"]
    A3["5-6 Diagnostics & Docs"]
    A4["7-9 Analysis / Proposal / Gap"]
    A5["10 Approval Gate"]
    A6["11-12 Remediation & Proof"]
    A7["13-14 Client Notify → Close"]
    A1 --> A2 --> A3 --> A4 --> A5 --> A6 --> A7
```

## 5 | Implementation Roadmap

| Phase | Timeline | Deliverables |
|-------|----------|--------------|
| **Week 1** | - Build custom MCP server- Confluence discovery → ChromaDB ingest | P-1 user story satisfied |
| **Week 2** | - LangGraph e2e test on 2 Jira tickets- Slack notifications & gap detection | P-2 & P-3 satisfied |
| **Month 2** | - Swap ChromaDB → Weaviate- Add PostgreSQL analytics tables- Real-time runbook webhooks | Prod-1 & Prod-2 |
| **Month 3** | - Effectiveness reports- HA deployment (Weaviate cluster, PG replication) | Prod-3 & Prod-4 |

# 6 | User Story 1: Basic Workflow Orchestration (Detailed Tasks)

## User Story Definition

**As a** MC-DBA engineer responding to database incidents  
**I want** a LangGraph workflow that automatically processes incident tickets and finds relevant runbooks  
**So that** I receive structured runbook recommendations without manual searching  

## Task Breakdown

| Task | Goal | Timeline | Success Metric |
|------|------|----------|---------------|
| **Task 1** | Core Workflow Infrastructure Setup | Week 1, Days 1-2 | Workflow compiles and MCP connects |
| **Task 2** | Incident Fetching & Processing Node | Week 1, Days 3-4 | Fetches incident data from test tickets |
| **Task 3** | Runbook Search & Results Integration | Week 1, Day 5 - Week 2, Day 1 | Returns top 3 relevant runbooks via ChromaDB |
| **Task 4** | End-to-End Workflow Execution & Testing | Week 2, Days 2-3 | End-to-end execution  B["WorkflowState Definition"]


### Task 1: Workflow Infrastructure Setup
```mermaid
flowchart TD
    A["LangGraph StateGraph"] --> B["WorkflowState Definition"]
    B --> C["GraphMCP Client Init"]
    C --> D["Atlassian MCP Connection"]
    D --> E["Basic START → END Flow"]
```

### Task 3: ChromaDB Integration
```mermaid
flowchart TD
    A["Incident Data"] --> B["Query Construction(summary + description)"]
    B --> C["ChromaDB Semantic Search"]
    C --> D["Client-Specific Filtering(AAVA/MCDBA spaces)"]
    D --> E["Top 3 Runbooks"]
    E --> F["Format for Jira Comment"]
```

## Acceptance Criteria

**Overall User Story Success Metrics**:
- ✅ LangGraph workflow processes incidents automatically
- ✅ MCP integration fetches Jira ticket data reliably
- ✅ Semantic search returns top 3 relevant runbooks
- ✅ End-to-end execution completes in under 30 seconds
- ✅ Basic error handling prevents workflow failures
- ✅ State management maintains context across nodes

## Implementation Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Workflow Engine** | LangGraph + GraphMCP | Orchestrate incident processing |
| **Vector Storage** | ChromaDB (local, persistent) | Semantic runbook search |
| **MCP Integration** | Atlassian MCP Server | Jira operations |
| **Embeddings** | OpenAI text-embedding-3-large | RAG-compatible vectors |
| **State Management** | WorkflowState (LangGraph) | Maintain context across nodes |

## Expected Outcome

After completing all 4 tasks, User Story 1 will deliver a working prototype that:
- Automatically processes Jira incidents from NESMCI, HEMCI, BMC, GROMC projects
- Searches ChromaDB for relevant runbooks from AAVA/MCDBA Confluence spaces
- Returns top 3 runbook recommendations with relevance scores
- Executes end-to-end workflow in under 30 seconds
- Provides foundation for Slack notifications (User Story 2) and gap detection (User Story 3)


## 7 | Expandability Highlights

* **Strategy Pattern:** any layer (discovery, vector, storage) can be hot-swapped without touching workflow logic.  
* **GraphMCP Nodes:** each workflow step is a first-class node—new paths (e.g., PagerDuty alerts) are added declaratively.  
* **RAG-Ready Embeddings:** embeddings use OpenAI *text-embedding-3-large* (1536-D, cosine norm) ensuring drop-in compatibility with future RAG engines.

## 8 | Prototype Definition of Done

1. **Runbooks discovered** from AAVA & MCDBA, hierarchy preserved on disk.  
2. **Two Jira tickets** processed end-to-end with top-3 runbook links added.  
3. **Slack alert** shows incident + runbook list within 5 s of comment.  
4. **Unit tests** cover discovery, embedding, search, and workflow routing.  
5. **Replaceability demo:** swap ChromaDB→mock store in <15 min with no workflow edits.

## 9 | Technical Stack Summary

| Component | Prototype | Production |
|-----------|-----------|------------|
| **Workflow Engine** | LangGraph + GraphMCP | LangGraph + GraphMCP |
| **Vector Storage** | ChromaDB (local) | Weaviate (distributed cluster) |
| **Runbook Discovery** | Confluence CQL search | Confluence + webhooks |
| **Persistence** | Files + metadata | PostgreSQL + audit trail |
| **Embeddings** | OpenAI text-embedding-3-large | OpenAI text-embedding-3-large |
| **MCP Integration** | Atlassian, Slack servers | Atlassian, Slack, Custom servers |

This plan delivers a lean prototype that can evolve—step-by-step—into a full production system while keeping every component replaceable and future-proof.