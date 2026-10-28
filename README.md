Supplier Intelligence RAG Platform
Enterprise Supplier Information, Sourcing, Risk, Performance & Supply Chain Intelligence

Architecture and Engineering Blueprint | Java · Spring Boot · PostgreSQL · pgvector · Vue.js · TypeScript · MCP · AWS EKS

!Architecture status !Backend !Frontend !Database !AI !Deployment

Table of Contents
Executive Summary
Business Capabilities
Architecture Principles
High-Level Architecture
Technology Stack
Repository Structure
Data Architecture and PostgreSQL
RAG Pipeline
Configurable AI Harness
Dynamic Chunking and Embedding
Explainable AI
Fine-Tuning Strategy
MCP and Enterprise Integrations
Security and Guardrails
Evaluation and Testing Framework
Caching and Performance
Local Development Setup
How to Run and Test
Kubernetes Deployment
AWS Production Deployment
Observability and Operations
CI/CD and MLOps
Implementation Roadmap
Definition of Done
References
1. Executive Summary

The Supplier Intelligence RAG Platform is an enterprise conversational AI system that enables procurement, sourcing, supply-chain, finance, and risk teams to ask natural-language questions about suppliers and receive grounded, explainable, and permission-aware answers.

The platform combines:

Structured enterprise data: Supplier information management, sourcing, risk, performance, ERP data, and supply-chain data from PostgreSQL.
Enterprise knowledge: Confluence, Jira, Word, PDF, and Excel documents.
Enterprise APIs: Internal REST APIs, GraphQL APIs, and a custom Model Context Protocol (MCP) server.
Retrieval-augmented generation: Hybrid retrieval, metadata filtering, reranking, contextual synthesis, and source citations.
Configurable AI orchestration: Model selection, reasoning budgets, chunking, embedding, caching, guardrails, evaluation, retries, and bounded agent loops.
Explainability: Evidence citations, source provenance, data freshness, retrieval diagnostics, and traceable answer generation.
Deployment flexibility: Local development, Docker, Kubernetes, and AWS.
1.1 Architectural objective

Build a system that can answer questions such as:

Which suppliers present the highest operational and financial risk?
Compare supplier performance across quality, delivery, cost, and compliance.
Which alternative suppliers can replace a supplier with a high disruption risk?
Explain why a supplier's risk score increased over the previous quarter.
Find sourcing opportunities based on supplier capabilities, location, lead time, and historical performance.
Summarize unresolved supplier-related Jira issues and relevant Confluence policies.
Combine ERP delivery records with supplier contracts, quality reports, and risk assessments.
Explain which source records support a recommendation and which information is missing.
1.2 Non-negotiable architectural decisions
Concern	Decision
Authoritative business data	PostgreSQL and approved enterprise systems
Semantic retrieval	PostgreSQL with pgvector
Structured questions	SQL and approved domain services
Document questions	RAG with citations
Cross-source questions	Hybrid orchestration over SQL, RAG, and approved tools
Backend	Java and Spring Boot
Frontend	Vue.js and TypeScript
LLM integration	Provider-neutral adapter architecture
Enterprise integrations	Atlassian MCP, custom MCP, REST, and GraphQL
Explainability	Evidence ledger and traceable execution plan
Configuration	Versioned, validated harness profiles
Evaluation	Offline datasets, regression tests, online quality metrics
Deployment	Containers, Kubernetes, AWS EKS
Security	Identity-aware access, tenant isolation, least privilege
AI safety	Bounded loops, tool allowlists, output validation, and human approval

Important: The LLM must not become the source of truth for supplier facts. Authoritative business values must be retrieved from trusted data sources and explicitly distinguished from generated interpretation.

2. Business Capabilities
2.1 Functional scope
Domain	Capabilities	Primary source
Supplier information management	Profiles, hierarchy, capabilities, certifications, contacts, locations	PostgreSQL
Supplier sourcing	Candidate discovery, qualification, comparison, sourcing rationale	PostgreSQL + documents
Supplier risk	Financial, operational, geographic, compliance, concentration, disruption risk	PostgreSQL + approved external or internal evidence
Supplier performance	Quality, delivery, cost, service-level, defect and incident trends	PostgreSQL
ERP integration	Purchase orders, invoices, receipts, spend, lead times, inventory	ERP-integrated PostgreSQL data
Supply-chain intelligence	Dependencies, bottlenecks, alternate suppliers, material exposure	PostgreSQL + supply-chain services
Enterprise knowledge	Policies, meeting notes, technical specifications, decisions	Confluence
Engineering and operations	Supplier incidents, defects, action items, project issues	Jira
Document intelligence	Contracts, reports, questionnaires, scorecards	Word, PDF, Excel
API intelligence	Approved business operations and queries	REST and GraphQL
Conversational AI	Natural-language questions, follow-ups, comparisons, summaries	AI orchestration harness
Explainability	Citations, evidence, data freshness, execution trace	Evidence service
Administration	Model configuration, chunking, evaluation, guardrails and access policies	Admin console
2.2 Example user interaction

User: Why is Supplier A considered high risk, and which qualified alternatives exist?

The agent should:

Identify the requested supplier and business context.
Retrieve current risk, performance, spend, and sourcing data from authorized services.
Retrieve relevant contracts, risk assessments, and supplier policies.
Retrieve relevant Jira incidents and Confluence documentation when authorized.
Compare evidence dates, confidence, and source reliability.
Determine which alternative suppliers satisfy the required qualifications.
Produce a grounded explanation with citations and clearly stated limitations.
Record the execution trace and quality metrics.

The answer should distinguish verified facts from interpretations and recommendations. It must not fabricate risk scores, supplier qualifications, or evidence.

3. Architecture Principles
3.1 Separate business truth from semantic knowledge

Structured business questions should use SQL or approved domain services. Documents should use vector and lexical retrieval. Questions spanning both should use a controlled orchestration plan.

Do not embed every ERP record and then depend on semantic search for exact amounts, counts, or current status.

3.2 Keep the AI harness provider-neutral

The application must not depend on a single model provider. Model access should be implemented behind stable Java interfaces, with provider-specific implementations and capability checks.

3.3 Make every important decision observable

Record the configuration version, retrieval strategy, selected sources, model provider, latency, token usage, tool invocations, guardrail results, and evaluation outcomes.

Avoid logging raw sensitive prompts, full documents, credentials, or unnecessary personal data.

3.4 Treat agent execution as a bounded workflow

The agent may retrieve, inspect, compare, and revise a response. It must not enter an unlimited reasoning or tool-calling loop.

Set explicit limits on iterations, wall-clock time, tool calls, token consumption, and monetary cost.

3.5 Use evidence-based explainability

Explainability means showing the supporting records, citations, calculations, and decision criteria. It does not mean exposing private internal chain-of-thought.

3.6 Version everything that changes answer quality

Version prompts, model configurations, embedding models, chunking strategies, index schemas, retrieval weights, rerankers, evaluation datasets, and harness profiles.

A change to any of these can change production behavior and must be tested.

4. High-Level Architecture
4.1 Logical architecture diagram
mermaid
flowchart TB
    U[Procurement and Supply Chain Users]
    UI[Vue.js and TypeScript UI]
    GW[Spring Boot API Gateway and BFF]
    AUTH[Identity and Access Control]
    AGENT[Conversational Agent Service]
    HARNESS[Configurable AI Harness]

    SQL[Structured Data Query Service]
    RAG[Document Retrieval Service]
    MCP[Enterprise MCP Client]
    API[REST and GraphQL Connectors]

    PG[(PostgreSQL)]
    VEC[(pgvector)]
    CACHE[(Redis - Optional)]
    DOCS[(S3 Document Store)]
    IDX[Document Ingestion and Indexing]

    LLM[LLM Provider Router]
    OLLAMA[Ollama]
    ANTHROPIC[Claude API]
    OPENAI[OpenAI API]

    EVAL[Evaluation and Feedback Service]
    OBS[OpenTelemetry and Observability]

    U --> UI --> GW
    GW --> AUTH
    GW --> AGENT
    AGENT --> HARNESS

    HARNESS --> SQL
    HARNESS --> RAG
    HARNESS --> MCP
    HARNESS --> API
    HARNESS --> LLM

    SQL --> PG
    RAG --> VEC
    RAG --> PG
    IDX --> DOCS
    IDX --> VEC
    IDX --> PG

    LLM --> OLLAMA
    LLM --> ANTHROPIC
    LLM --> OPENAI

    HARNESS --> CACHE
    HARNESS --> EVAL
    GW --> OBS
    HARNESS --> OBS
    IDX --> OBS

4.2 Architectural layers

Experience layer

Vue.js conversational interface.
Source citations and expandable evidence.
Supplier comparison tables and trend charts.
Conversation history with appropriate access controls.
Optional administration interface for harness configuration.

Application layer

Spring Boot API.
Authentication and authorization.
Conversation and session management.
Query classification and execution planning.
Domain services and API contracts.

Intelligence layer

Configurable harness.
Hybrid retrieval.
Query rewriting and decomposition.
Reranking and evidence fusion.
LLM routing, response synthesis, and output validation.

Data and integration layer

PostgreSQL domain data.
pgvector embeddings.
Document storage.
Atlassian MCP.
Custom MCP server.
Internal REST and GraphQL clients.

Operations layer

Evaluation datasets and regression testing.
Metrics, traces, and structured logs.
Model and prompt versioning.
Configuration management.
Security audit events and feedback workflows.
4.3 Recommended deployment boundaries

Start with a modular monolith for the primary Spring Boot application. Separate document ingestion and evaluation workers into independently deployable processes when operational requirements justify it.

Avoid splitting every service into a microservice at the beginning. Retrieval, query planning, and response synthesis have closely coupled data and latency requirements.

Recommended initial deployables:

supplier-ai-api — conversational and administration APIs.
supplier-ai-ingestion — document ingestion, parsing, chunking, and indexing.
supplier-ai-evaluator — scheduled and CI evaluation jobs.
supplier-ai-ui — Vue.js application.
supplier-ai-mcp — custom MCP server, if it requires a separate security boundary.
5. Technology Stack
Layer	Recommended technology
Language	Java 21 LTS baseline; validate against the selected Spring release
Backend	Spring Boot 3.x-compatible baseline, or a newer supported release after compatibility validation
Dependency management	Maven
Security	Spring Security, OAuth 2.0/OIDC
API	REST, OpenAPI
Frontend	Vue 3, TypeScript, Vite
Frontend testing	Vitest, Vue Test Utils, Playwright
Database	PostgreSQL
Vector search	pgvector
Database migrations	Flyway
Embeddings	Configurable local or hosted embedding provider
LLMs	Ollama, Anthropic Claude API, OpenAI API
Enterprise integration	Atlassian MCP, custom MCP, REST and GraphQL
Document parsing	Apache Tika and format-specific parsers
PDF extraction	PDFBox or another maintained PDF parser
Word and Excel	Apache POI or equivalent maintained parsers
Cache	Caffeine locally; Redis when shared caching is required
Resilience	Timeouts, retries, circuit breakers and bulkheads
Observability	OpenTelemetry, Micrometer, Prometheus, Grafana
Logs	Structured JSON logs and centralized log aggregation
Containers	Docker
Orchestration	Kubernetes
AWS	EKS, RDS for PostgreSQL, S3, ECR, Secrets Manager, CloudWatch
CI/CD	GitHub Actions or the organization's approved pipeline
IaC	Terraform
Evaluation	Custom evaluation harness with versioned datasets and optional external evaluation tooling
Model-provider clarification
Ollama: Local model inference. Suitable for development, privacy-sensitive experimentation, and self-hosted inference when hardware and model licensing permit.
Anthropic Claude: Access through the supported Claude API and an appropriate provider adapter.
OpenAI: Use the OpenAI API for application inference. ChatGPT product subscriptions and API billing are separate.
Embeddings: Choose a compatible embedding model separately where necessary. A chat model is not automatically an embedding model.

Pin compatible dependency versions and container images. Verify licensing, model availability, provider-specific structured-output support, tool calling, embedding dimensions, and regional data-processing requirements before production deployment.

6. Repository Structure

Recommended monorepo layout:

text
supplier-intelligence-rag/
├── README.md
├── pom.xml
├── .env.example
├── docker-compose.yml
├── Makefile
│
├── apps/
│   ├── supplier-ai-api/
│   ├── supplier-ai-ingestion/
│   ├── supplier-ai-evaluator/
│   └── supplier-ai-mcp/
│
├── frontend/
│   └── supplier-ai-ui/
│
├── libs/
│   ├── domain-model/
│   ├── data-access/
│   ├── rag-core/
│   ├── ai-harness/
│   ├── llm-providers/
│   ├── embedding-providers/
│   ├── retrieval/
│   ├── document-processing/
│   ├── enterprise-connectors/
│   ├── security-policy/
│   ├── evaluation/
│   └── observability/
│
├── config/
│   ├── application.yml
│   ├── harness/
│   │   ├── development.yml
│   │   ├── balanced.yml
│   │   ├── high-accuracy.yml
│   │   └── low-latency.yml
│   ├── prompts/
│   ├── chunking/
│   ├── guardrails/
│   └── evaluations/
│
├── database/
│   ├── migrations/
│   ├── seed/
│   └── test-fixtures/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   ├── retrieval/
│   ├── security/
│   ├── evaluation/
│   └── performance/
│
├── deploy/
│   ├── docker/
│   ├── kubernetes/
│   ├── helm/
│   └── terraform/
│
├── scripts/
│   ├── bootstrap-local.sh
│   ├── load-demo-data.sh
│   ├── ingest-documents.sh
│   └── run-evaluations.sh
│
└── docs/
    ├── architecture.md
    ├── threat-model.md
    ├── data-governance.md
    ├── harness-configuration.md
    ├── evaluation-strategy.md
    ├── deployment-runbook.md
    └── adr/


This is the target repository structure; each directory becomes real after the corresponding module is implemented. Do not introduce empty modules simply to make the tree look complete.

7. Data Architecture and PostgreSQL
7.1 Separate the business-data and knowledge domains

Use PostgreSQL for structured supplier data and pgvector for document embeddings. They may share a PostgreSQL cluster initially, but separate schemas, roles, access policies, and resource limits should be used.

Suggested schemas:

Schema	Purpose
supplier	Supplier master, contacts, capabilities, certifications
sourcing	Sourcing events, bids, qualification and award data
risk	Risk factors, assessments, incidents and risk history
performance	Quality, delivery, cost and service metrics
erp	Approved ERP extracts or integration views
supply_chain	Materials, facilities, dependencies and lead times
knowledge	Document metadata, versions, chunks and embeddings
ai	Conversations, traces, configurations and evaluation runs
audit	Access and administrative audit events

Use domain-specific ownership and migrations rather than allowing the LLM layer to modify business tables directly.

7.2 Example domain model
mermaid
erDiagram
    SUPPLIER ||--o{ SUPPLIER_CAPABILITY : has
    SUPPLIER ||--o{ SUPPLIER_CERTIFICATION : holds
    SUPPLIER ||--o{ RISK_ASSESSMENT : receives
    SUPPLIER ||--o{ PERFORMANCE_METRIC : records
    SUPPLIER ||--o{ SOURCING_BID : submits
    SUPPLIER ||--o{ PURCHASE_ORDER : receives

    SUPPLIER {
        uuid supplier_id PK
        string supplier_code
        string legal_name
        string status
        string country_code
        timestamp updated_at
    }

    RISK_ASSESSMENT {
        uuid assessment_id PK
        uuid supplier_id FK
        string risk_category
        decimal risk_score
        string methodology_version
        timestamp assessed_at
    }

    PERFORMANCE_METRIC {
        uuid metric_id PK
        uuid supplier_id FK
        string metric_type
        decimal metric_value
        date period_start
        date period_end
    }


The diagram is illustrative. Adapt entities and relationships to the existing ERP and procurement data model instead of creating a second, inconsistent supplier master.

7.3 Document and chunk model

A useful document model includes:

document_id
source_system
source_document_id
source_uri
title
document_type
supplier_id, when applicable
tenant_id
classification
acl_metadata
source_version
content_hash
ingestion_status
created_at
updated_at
indexed_at

Each chunk should include:

chunk_id
document_id
chunk_index
content
embedding
embedding_model
embedding_version
chunking_strategy
chunking_version
token_count
page_number or source-specific location
section_path
source_version
metadata

Preserve document and chunk provenance so that each answer can point to the original source location.

7.4 Example pgvector schema

The following is illustrative SQL. Choose the vector dimension based on the selected embedding model and validate it before applying the migration.

sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE SCHEMA IF NOT EXISTS knowledge;

CREATE TABLE knowledge.document (
    document_id UUID PRIMARY KEY,
    source_system TEXT NOT NULL,
    source_document_id TEXT NOT NULL,
    title TEXT NOT NULL,
    source_uri TEXT,
    content_hash TEXT NOT NULL,
    source_version TEXT,
    tenant_id TEXT NOT NULL,
    classification TEXT NOT NULL DEFAULT 'INTERNAL',
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    indexed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (tenant_id, source_system, source_document_id)
);

CREATE TABLE knowledge.document_chunk (
    chunk_id UUID PRIMARY KEY,
    document_id UUID NOT NULL
        REFERENCES knowledge.document(document_id)
        ON DELETE CASCADE,
    chunk_index INTEGER NOT NULL,
    content TEXT NOT NULL,
    embedding VECTOR(1536),
    embedding_model TEXT NOT NULL,
    chunking_version TEXT NOT NULL,
    token_count INTEGER,
    page_number INTEGER,
    section_path TEXT,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (document_id, chunk_index, chunking_version)
);

CREATE INDEX document_tenant_source_idx
    ON knowledge.document (tenant_id, source_system);

CREATE INDEX document_chunk_document_idx
    ON knowledge.document_chunk (document_id);

CREATE INDEX document_chunk_metadata_idx
    ON knowledge.document_chunk USING GIN (metadata);


VECTOR(1536) is an example, not a universal setting. Replace it if the chosen embedding model uses a different dimension. When multiple incompatible embedding dimensions or model families are required, use separate tables, indexes, or explicitly versioned storage designs.

For a production deployment, benchmark pgvector's HNSW or IVFFlat indexes using the actual workload, filtering patterns, data volume, and recall targets. Index selection and parameters should be evidence-driven.

7.5 Structured data query strategy

Use a dedicated supplier domain service for high-value business questions:

text
SupplierQueryService
├── findSupplierProfile()
├── compareSupplierPerformance()
├── getSupplierRiskHistory()
├── findQualifiedAlternatives()
├── getSupplierSpend()
├── analyzeDeliveryPerformance()
└── getSupplyChainDependencies()


Recommended execution model:

Classify the question.
Map it to an approved domain operation.
Validate parameters and authorization.
Execute parameterized SQL or call the domain API.
Normalize the result into a typed response.
Record the data timestamp and provenance.
Pass only relevant, authorized results to the synthesis stage.

For more exploratory questions, a constrained SQL planner can be added later. It should use read-only database roles, schema and table allowlists, query timeouts, row limits, tenant filters, and query validation.

Never provide unrestricted production SQL access to an LLM.

7.6 Freshness and consistency

Not every source has the same freshness:

Supplier master data may be synchronized frequently.
Risk scores may be refreshed daily or on an event.
Jira issues may change continuously.
Confluence content may be indexed asynchronously.
Historical performance metrics may be calculated in batch.

Include as_of, updated_at, or equivalent freshness metadata in query results. When source timestamps conflict, explain the difference rather than silently combining stale and current values.

8. RAG Pipeline
8.1 Ingestion pipeline
mermaid
flowchart TD
    SRC[PostgreSQL and Enterprise Sources]
    DISC[Discovery and Change Detection]
    FETCH[Fetch and Access Validation]
    PARSE[Format-Aware Parsing]
    CLEAN[Normalization and Redaction]
    SPLIT[Dynamic Chunking]
    EMBED[Embedding Generation]
    STORE[Metadata and Vector Storage]
    VERIFY[Index Verification]
    READY[Available for Retrieval]

    SRC --> DISC --> FETCH --> PARSE
    PARSE --> CLEAN --> SPLIT --> EMBED
    EMBED --> STORE --> VERIFY --> READY


Recommended processing stages:

Discover documents or source changes.
Fetch content using approved connector identities.
Enforce document access and classification policies.
Parse Word, PDF, Excel, Confluence, and Jira content.
Normalize text while preserving structural metadata.
Detect duplicates and content changes using hashes.
Select a chunking strategy.
Generate embeddings.
Write chunks and metadata transactionally where possible.
Update source-version and indexing status.
Validate retrieval and indexing metrics.
Make the new index version available.

Do not overwrite a valid index with a partial ingestion result. Prefer staged indexing and controlled activation.

8.2 Query-time pipeline
mermaid
flowchart TD
    Q[User Question]
    AUTH[Identity and Policy Checks]
    INTENT[Intent and Entity Resolution]
    PLAN[Execution Plan]
    STRUCT[Structured Data Retrieval]
    RETRIEVE[Lexical and Vector Retrieval]
    FILTER[ACL and Metadata Filtering]
    RERANK[Evidence Reranking]
    FUSE[Evidence Fusion]
    GENERATE[LLM Response Generation]
    VALIDATE[Grounding and Guardrail Validation]
    ANSWER[Answer with Citations]
    TRACE[Trace and Feedback]

    Q --> AUTH --> INTENT --> PLAN
    PLAN --> STRUCT
    PLAN --> RETRIEVE
    RETRIEVE --> FILTER --> RERANK
    STRUCT --> FUSE
    RERANK --> FUSE
    FUSE --> GENERATE --> VALIDATE --> ANSWER
    ANSWER --> TRACE

8.3 Retrieval strategy

Use a hybrid retrieval architecture:

Vector search: Conceptual and semantic similarity.
Lexical search: Exact supplier names, codes, purchase order numbers, issue keys, and policy identifiers.
Metadata filtering: Tenant, user permissions, supplier, document type, classification, and freshness.
Reranking: Improve ordering of retrieved evidence using a reranker where justified.
Evidence fusion: Combine structured query results and document passages.
Deduplication: Avoid repeated content from duplicated or superseded documents.

A recommended retrieval sequence is:

Normalize the question without losing identifiers.
Resolve supplier names, IDs, dates, and other entities.
Generate one or more retrieval queries.
Execute authorized structured queries and document searches in parallel when safe.
Apply access controls before content reaches the model.
Rerank the eligible evidence.
Build a compact evidence package.
Generate a cited answer.
Validate citations and claims before returning the result.
8.4 Query routing

Use explicit routing policies instead of sending every question to the vector database.

Query type	Preferred execution
Exact supplier profile	Structured query
Supplier spend by quarter	SQL or domain API
Supplier contract obligations	Document RAG
Supplier incident summary	Jira connector or MCP
Sourcing alternatives	Structured filtering plus evidence retrieval
Risk explanation	Structured risk data plus supporting documents
Multi-source comparison	Parallel retrieval and evidence fusion
Unsupported or ambiguous question	Ask a clarifying question or disclose limitations
8.5 Answer contract

A successful response should contain:

Direct answer.
Supporting evidence and source links.
Calculations and comparison criteria, where relevant.
Data freshness or as_of timestamps.
Material assumptions and unresolved conflicts.
Explicit uncertainty when evidence is insufficient.

Every citation should refer to a real retrieved source. Citation validation must verify that the cited document or record exists and was actually included in the evidence supplied to the model.

9. Configurable AI Harness

The AI harness is the platform's primary orchestration capability. It coordinates retrieval, model invocation, reasoning workflows, tool execution, validation, caching, and evaluation.

9.1 Harness responsibilities
text
AI Harness
├── RequestContextBuilder
├── IdentityAndPolicyResolver
├── IntentClassifier
├── EntityResolver
├── QueryPlanner
├── ModelRouter
├── StructuredDataExecutor
├── RetrievalExecutor
├── MCPToolExecutor
├── EvidenceAssembler
├── RerankingService
├── PromptComposer
├── ResponseGenerator
├── CitationValidator
├── GuardrailEngine
├── BoundedRetryController
├── CacheCoordinator
├── EvaluationHooks
└── TraceRecorder


Each component should have a defined interface, observable behavior, and test coverage.

9.2 Harness configuration

The following is a reference configuration format, not a complete implementation:

yaml
harness:
  profile: balanced
  version: "1.0"

  execution:
    max_iterations: 4
    max_tool_calls: 8
    request_timeout_ms: 30000
    max_total_tokens: 12000
    max_retries: 2
    allow_parallel_retrieval: true

  routing:
    structured_data: domain_service
    document_retrieval: hybrid
    reranking: enabled
    multi_source_fusion: enabled

  retrieval:
    dense_top_k: 40
    lexical_top_k: 40
    rerank_top_k: 10
    final_context_top_k: 8
    minimum_relevance_score: 0.35

  chunking:
    strategy: structure_aware
    target_tokens: 700
    overlap_tokens: 100
    max_tokens: 1000
    preserve_tables: true
    preserve_section_headers: true

  caching:
    query_cache_enabled: true
    embedding_cache_enabled: true
    response_cache_enabled: false
    ttl_seconds: 300

  guardrails:
    enforce_document_acl: true
    enforce_tool_allowlist: true
    validate_citations: true
    detect_prompt_injection: true
    block_unauthorized_sql: true
    require_grounding: true

  evaluation:
    collect_trace: true
    sample_online_evaluations: true
    enable_feedback: true

  llm:
    planner: local-planner
    synthesis: primary-synthesis
    fallback: local-fallback


The values are starting points for experimentation, not universal production defaults. Tune them using representative supplier questions, actual document lengths, latency budgets, and measured evaluation results.

9.3 Configuration precedence

Recommended precedence:

Immutable security policies and hard system limits.
Environment-specific settings.
Approved tenant or user permissions.
Versioned harness profile.
Request-level preferences permitted by policy.

Users must not be able to override authorization, disable citation validation, expand tool permissions, or bypass token and iteration limits through conversational instructions.

9.4 Java interfaces

Illustrative interfaces:

java
public interface LlmProvider {
    LlmResponse generate(LlmRequest request);
}

public interface EmbeddingProvider {
    List<float[]> embed(List<String> texts);
}

public interface RetrievalService {
    RetrievalResult retrieve(RetrievalRequest request);
}

public interface StructuredQueryService {
    StructuredResult execute(StructuredQueryRequest request);
}

public interface ToolExecutionService {
    ToolResult execute(ApprovedToolCall toolCall);
}

public interface HarnessEngine {
    AgentResponse execute(AgentRequest request);
}


For production, use explicit asynchronous variants where appropriate, typed exceptions, cancellation, deadline propagation, tracing context, and provider-specific capability metadata.

Do not make a synchronous interface the only implementation path if ingestion or multi-source retrieval requires asynchronous execution.

9.5 Model routing policy

Model routing should consider:

Task type.
Required tool-calling or structured-output capabilities.
Context-window requirements.
Data classification.
Provider availability.
Latency and cost budgets.
Quality measured by evaluations.
Regional processing restrictions.
Whether the task requires a local model.

A more capable model is not automatically the best choice for every task. Use a smaller, cheaper model for straightforward classification when it meets quality requirements; reserve more expensive models for complex synthesis when justified.

Fallbacks must be explicit and compatible with the task. Do not silently fall back to a model that violates data residency or privacy policies.

10. Dynamic Chunking and Embedding
10.1 Chunking requirements

A single fixed-size chunking strategy is rarely ideal for every document format.

Source	Recommended initial strategy
Confluence	Heading-aware semantic chunks
Jira	Issue-level chunks with fields and comments separated
Word	Heading, paragraph, table and section-aware chunks
PDF	Page-aware chunks with layout and table preservation
Excel	Sheet, table, row-group and column-header-aware chunks
Contracts	Clause and section-aware chunks
ERP records	Structured records; avoid indiscriminate text chunking
Technical reports	Section-aware chunks with controlled semantic splitting
10.2 Configurable chunking algorithm

The chunking engine should consider:

Document format.
Section boundaries.
Token count.
Table boundaries.
Semantic similarity.
Metadata and access labels.
Overlap requirements.
Maximum context length.
Embedding model limits.

Reference algorithm:

text
Input document
    |
    v
Parse into structured elements
    |
    v
Preserve headings, tables, pages and source locations
    |
    v
Select chunking profile by document type
    |
    v
Split at structural boundaries
    |
    v
Split oversized sections by token budget
    |
    v
Apply controlled overlap where useful
    |
    v
Attach metadata and provenance
    |
    v
Generate embeddings and validate dimensions
    |
    v
Store versioned chunks

10.3 Dynamic configuration example
yaml
chunking_profiles:
  confluence:
    strategy: structure_aware
    target_tokens: 650
    overlap_tokens: 80
    preserve_headings: true

  jira:
    strategy: issue_aware
    target_tokens: 500
    overlap_tokens: 50
    separate_comments: true

  pdf:
    strategy: page_and_section_aware
    target_tokens: 800
    overlap_tokens: 100
    preserve_tables: true

  word:
    strategy: heading_aware
    target_tokens: 750
    overlap_tokens: 100
    preserve_tables: true

  excel:
    strategy: table_aware
    max_rows_per_chunk: 50
    repeat_column_headers: true


These are example profiles. Validate them against the actual document corpus.

10.4 Embedding strategy

Maintain an embedding registry with:

Provider.
Model identifier and version.
Vector dimensions.
Normalization behavior.
Supported languages.
Maximum input length.
Cost and latency measurements.
Evaluation results.
Index compatibility.

Do not mix embeddings from incompatible models in one vector space without an explicitly validated strategy.

When changing embedding models, use versioned reindexing:

Create a new index or chunk version.
Re-embed a representative corpus.
Run retrieval evaluations.
Compare relevance, latency, and storage costs.
Activate the new version.
Retain rollback capability.
Remove old embeddings only after the retention policy allows it.
11. Explainable AI
11.1 Explainability objectives

The system should answer four questions:

What did the agent conclude?
Which data and documents support the conclusion?
How fresh and reliable is the evidence?
What assumptions, conflicts, or limitations could affect the result?
11.2 Evidence ledger

Maintain an evidence ledger for every completed request.

Example:

json
{
  "request_id": "req-example",
  "answer_version": "answer-v1",
  "evidence": [
    {
      "evidence_id": "ev-001",
      "source_type": "postgresql",
      "source_record": "risk_assessment:example",
      "source_timestamp": "2026-10-27T12:00:00Z",
      "claim": "The supplier has an elevated operational risk score",
      "retrieval_method": "structured_query"
    },
    {
      "evidence_id": "ev-002",
      "source_type": "confluence",
      "source_record": "document-reference",
      "source_timestamp": "2026-10-25T09:00:00Z",
      "claim": "A mitigation policy applies to this supplier category",
      "retrieval_method": "hybrid_search"
    }
  ],
  "limitations": [
    "Financial data freshness must be verified"
  ]
}


This is an illustrative record. Production identifiers must refer to real records, and sensitive source content should not be copied unnecessarily into trace storage.

11.3 Frontend evidence experience

Each answer should support:

Inline citations.
Click-through to the original source when permitted.
Expandable source excerpts.
Supplier and document metadata.
Data freshness labels.
A clear distinction between facts, calculations, and recommendations.
A disclosure when evidence is missing or conflicting.
11.4 Risk and performance explainability

For a supplier risk explanation, display the underlying risk components only when they exist in the authoritative risk model.

For example:

text
Supplier Risk Explanation

Overall risk: High
Assessment timestamp: 2026-10-27

Contributing evidence:
- Delivery performance below the configured threshold
- An unresolved critical supplier incident
- A recent risk assessment requiring mitigation

Limitations:
- Financial exposure data is older than the preferred freshness window

Sources:
- Supplier risk assessment
- Delivery performance records
- Jira incident
- Approved risk policy


The example is illustrative; the platform must calculate and populate these values from actual source data.

Do not invent a numerical risk score, weighting methodology, causal relationship, or confidence value merely because the LLM can generate a plausible explanation.

12. Fine-Tuning Strategy

Fine-tuning should be an optional capability, not a prerequisite for delivering the initial RAG platform.

12.1 Recommended progression

Phase 1 — Prompt and retrieval optimization

Improve source selection.
Tune chunking and metadata filters.
Optimize prompt templates.
Add structured outputs and citation validation.
Establish evaluation datasets.

Phase 2 — Embedding and reranker optimization

Compare embedding models.
Evaluate domain-specific retrieval quality.
Train or fine-tune a compatible reranker when sufficient labeled data exists.
Measure recall and ranking improvements.

Phase 3 — Model fine-tuning

Consider supervised fine-tuning when a specific model-supported task has a measurable quality gap that prompt and retrieval improvements cannot adequately address.

Potential tasks include:

Supplier question classification.
Domain-specific intent routing.
Structured extraction.
Standardized response formatting.
Domain terminology and task behavior.

Fine-tuning does not replace current supplier data retrieval, access control, or citations. It should not be the mechanism used to keep supplier facts up to date.

12.2 Fine-tuning pipeline
text
Approved and de-identified examples
    |
    v
Data quality and licensing checks
    |
    v
Train / validation / test split
    |
    v
Baseline evaluation
    |
    v
Model or adapter training
    |
    v
Held-out evaluation and safety testing
    |
    v
Human review and approval
    |
    v
Versioned model registry
    |
    v
Canary rollout and rollback

12.3 Dataset governance

Every dataset should include:

Data source and lawful use basis.
Ownership and approval.
PII and confidential-data review.
Labeling guidelines.
Version and lineage.
Deduplication.
Train/validation/test separation.
Contamination checks.
Retention and deletion rules.

For local Ollama models, the availability of fine-tuning depends on the underlying model and training tooling. Training may require a separate GPU-enabled pipeline and exporting or packaging the resulting model in a compatible format.

Hosted models have provider-specific fine-tuning capabilities and restrictions. Validate those capabilities before committing to a training design.

12.4 Success criteria

A fine-tuned model should replace a baseline only when held-out evaluation demonstrates meaningful improvements without unacceptable regressions in grounding, safety, latency, cost, or operational complexity.

13. MCP and Enterprise Integrations
13.1 Integration architecture

Implement adapters rather than embedding vendor-specific logic throughout the harness.

text
EnterpriseConnectorRegistry
├── PostgreSqlDomainConnector
├── AtlassianMcpConnector
├── CustomMcpConnector
├── RestApiConnector
├── GraphqlApiConnector
├── WordDocumentConnector
├── PdfDocumentConnector
└── ExcelDocumentConnector

13.2 Atlassian MCP

Use an approved Atlassian MCP integration for the required Confluence and Jira operations.

Responsibilities include:

Authentication and authorization.
Tool discovery and capability mapping.
Search and retrieval.
Pagination and rate-limit handling.
Source URI and version preservation.
Error normalization.
Audit logging.
Permission-aware access.

Do not assume every Atlassian MCP implementation exposes the same tools or authentication flows. Validate the actual server's supported operations and deployment requirements.

Prefer read-only operations for the initial release. Mutating operations, such as creating Jira issues or editing Confluence pages, should require explicit authorization and appropriate user confirmation.

13.3 Custom MCP server

Build a custom MCP server when enterprise data or approved business operations need to be exposed through a standardized tool interface.

Example tools:

text
get_supplier_profile
search_suppliers
compare_supplier_performance
get_supplier_risk_assessment
get_supplier_risk_history
find_qualified_alternatives
get_supplier_spend_summary
get_supply_chain_dependencies
search_supplier_documents


Each tool should define:

Input schema.
Output schema.
Required permissions.
Read or write classification.
Timeout.
Maximum result size.
Audit requirements.
Data classification.
Error behavior.

MCP is a tool integration protocol, not an authorization system. The server and the underlying business service must enforce authorization independently.

13.4 Internal REST APIs

Use Spring's supported HTTP client facilities, such as RestClient or WebClient, depending on the application's synchronous and reactive requirements.

Implement:

OAuth 2.0 client credentials or another approved authentication method.
Explicit connection and request timeouts.
Retries for eligible transient failures only.
Circuit breakers and concurrency limits.
OpenAPI contract validation.
Request correlation IDs.
Redaction of sensitive fields in logs.
13.5 GraphQL APIs

Use an appropriate Java GraphQL client or a maintained Spring integration.

Controls should include:

Persisted queries or operation allowlists where appropriate.
Query depth and complexity limits.
Pagination.
Field-level authorization.
Response-size limits.
Timeout enforcement.
Error and partial-data handling.

Do not allow an LLM to construct unrestricted GraphQL queries against internal production APIs.

13.6 Word, PDF, and Excel ingestion

Word

Extract paragraphs, headings, tables, hyperlinks, and document metadata.
Preserve section hierarchy.
Handle DOCX and legacy DOC formats only when required.

PDF

Extract page-level text and tables.
Detect image-only or scanned documents.
Use OCR only when justified and approved.
Retain page references for citations.

Excel

Preserve workbook, sheet, table, column, and row context.
Repeat column headers in row-group chunks.
Normalize cell types and dates.
Treat formulas and displayed values carefully.
Apply size and parser limits to untrusted files.

Ingestion must be idempotent, version-aware, and resilient to malformed or oversized documents.

14. Security and Guardrails

Security is a foundational requirement because supplier contracts, pricing, financial exposure, and procurement decisions may be confidential.

14.1 Security controls
Control	Implementation
Authentication	Enterprise OIDC identity provider
Authorization	RBAC plus attribute-based policies where needed
Tenant isolation	Enforced in domain queries, retrieval filters, and storage
Document ACL	Source permissions mapped into retrieval authorization
Secrets	Environment-specific secret manager
Database	Least-privilege roles, TLS and encrypted storage
API protection	Rate limits, payload limits and timeouts
Audit	Security events and administrative change history
Tool security	Allowlisted operations with per-tool permissions
LLM data handling	Provider and model selection based on classification policy
Logging	Redaction and limited retention
Supply chain	Dependency scanning, image scanning and signed artifacts
14.2 Prompt injection

Treat documents, Jira descriptions, Confluence pages, and tool outputs as untrusted input. They may contain instructions that attempt to override system policies or trigger unauthorized actions.

Mitigations:

Separate instructions from retrieved content.
Clearly label retrieved material as untrusted evidence.
Never allow retrieved documents to change tool permissions.
Validate all tool calls against schemas and policy.
Apply access controls outside the model.
Restrict outbound network access.
Test adversarial document examples.
Require explicit confirmation for sensitive write operations.

A prompt alone is not an adequate security boundary.

14.3 Guardrail stages
text
Incoming request
    |
    v
Identity, tenant and authorization checks
    |
    v
Input size and policy validation
    |
    v
Query planning and tool allowlist
    |
    v
Permission-aware retrieval
    |
    v
Evidence and prompt-injection checks
    |
    v
Model generation
    |
    v
Citation, schema and policy validation
    |
    v
Safe response or controlled refusal

14.4 Data leakage prevention

Do not rely on vector similarity to enforce permissions.

Authorization must be enforced before evidence is sent to the model. For sensitive use cases, apply defense in depth using database access policies, tenant-aware queries, source-system permissions, and independently tested application-level authorization.

Cache keys must incorporate all relevant authorization context, data versions, and configuration versions. Never serve a cached response to a user who could not access the original evidence.

14.5 Human approval

Require explicit human approval for high-impact actions such as:

Supplier qualification or disqualification.
Sourcing award recommendations that trigger procurement workflows.
Changes to supplier master data.
Changes to risk assessments or official ratings.
Creation or modification of ERP transactions.

The agent may support these decisions, but it must not silently convert generated recommendations into authoritative business actions.

15. Evaluation and Testing Framework

Evaluation must be a first-class subsystem of the harness rather than a final testing step.

15.1 Evaluation categories
Category	Example metrics
Retrieval	Recall@k, Precision@k, MRR, nDCG
Grounding	Supported-claim rate, citation correctness
Answer quality	Correctness, completeness, relevance
Abstention	Unsupported-answer rate, appropriate refusal rate
Security	ACL leakage, prompt-injection success, unauthorized tool calls
Performance	p50/p95 latency, timeout rate
Reliability	Error rate, retry rate, provider fallback rate
Cost	Tokens per request, cost per successful answer
Operations	Index freshness, ingestion failures, cache hit rate

Metrics must be defined consistently. For example, a supported-claim rate should be calculated against a labeled set of answer claims, not an arbitrary model-generated confidence score.

15.2 Evaluation dataset

Create a versioned dataset containing:

User question.
User role and tenant context.
Expected answer facts.
Required evidence sources.
Relevant documents or record identifiers.
Expected citations.
Required structured queries or tools.
Acceptable abstention behavior.
Known adversarial examples.

Include examples from all major domains:

Supplier profiles.
Supplier performance.
Risk history.
Sourcing comparisons.
ERP and spend analytics.
Confluence and Jira.
Contracts and PDF reports.
Excel scorecards.
Multi-source and conflicting-data questions.
Permission-sensitive requests.
15.3 Example test case
json
{
  "id": "supplier-risk-001",
  "question": "Why has supplier SUP-100 risk increased?",
  "tenant": "test-tenant",
  "expected_tools": [
    "get_supplier_risk_history",
    "compare_supplier_performance",
    "search_supplier_documents"
  ],
  "required_evidence": [
    "risk_history",
    "performance_records"
  ],
  "must_cite_sources": true,
  "allow_abstention": true
}


The example represents a test specification. Actual expected results and identifiers should come from a controlled test database.

15.4 Evaluation gates

Run evaluations at the following points:

Before merging changes to prompts or retrieval logic.
Before changing embedding models.
Before enabling a new LLM provider or routing policy.
Before releasing a new harness configuration.
Before promoting a fine-tuned model.
After major source-system or schema changes.
Periodically against production-representative data.
15.5 Illustrative release thresholds

These are initial targets to calibrate against a labeled baseline, not universal guarantees.

Metric	Initial target
Retrieval Recall@10	At least 0.90
Citation correctness	At least 0.98
Supported-claim rate	At least 0.95
Unauthorized cross-tenant retrieval	0 tolerated
Unauthorized tool execution	0 tolerated
Regression in critical evaluation cases	0 tolerated without approval
Successful ingestion of valid fixtures	100%
API integration test pass rate	100%

High-risk workflows should have stricter acceptance criteria and explicit human review. A good average score cannot compensate for a critical access-control failure.

15.6 Test pyramid

Unit tests

Query classification.
Chunking and metadata generation.
Prompt assembly.
Citation validation.
Authorization policies.
Configuration validation.
Retry and loop limits.

Integration tests

PostgreSQL and pgvector.
Database migrations.
REST and GraphQL clients.
MCP tool contracts.
Document parsers.
Cache invalidation.

End-to-end tests

Login and authorization.
Conversational queries.
Multi-turn supplier comparisons.
Citation rendering.
Error and abstention behavior.

AI evaluations

Retrieval quality.
Grounded answer quality.
Adversarial behavior.
Model-provider regression.
Cost and latency.

Use deterministic fixtures and mock LLM responses for most CI tests. Run real-model evaluations in a separate controlled pipeline.

16. Caching and Performance
16.1 Cache layers
Cache	Purpose	Key considerations
Embedding cache	Avoid re-embedding unchanged text	Model version and normalized content hash
Retrieval cache	Reuse eligible search results	Query, filters, ACL, index version
Structured-query cache	Reduce repeated read queries	Tenant, parameters, freshness
Response cache	Reuse complete answers where safe	User access, source freshness, prompt and model versions
Metadata cache	Reduce repeated source lookups	TTL and invalidation
Provider connection pools	Reduce connection overhead	Concurrency and timeouts
16.2 Caching policy

Start with embedding and carefully scoped metadata caching. Enable query and response caching only after correctness and authorization tests.

A cached answer is unsafe if any of the following changes:

User permissions.
Tenant membership.
Source document version.
Supplier risk or performance data.
Prompt version.
Model configuration.
Harness policy.
Relevant business-data freshness window.

For risk, compliance, and sourcing decisions, prefer authoritative fresh queries or short, explicitly defined freshness windows.

16.3 Latency optimization

Measure latency by stage:

text
Request validation
  + entity resolution
  + query planning
  + structured retrieval
  + vector/lexical retrieval
  + reranking
  + generation
  + output validation


Useful optimizations include:

Parallelize independent retrieval tasks.
Limit evidence passed to the LLM.
Use targeted query decomposition.
Avoid unnecessary reasoning iterations.
Use asynchronous ingestion.
Reuse provider connections.
Apply request deadlines and cancellation.
Monitor pgvector index performance and query plans.

Do not optimize by dropping mandatory authorization checks or citation validation.

17. Local Development Setup
17.1 Prerequisites

Install:

Java 21.
Maven.
Node.js LTS and npm.
Docker and Docker Compose.
Git.
An Ollama installation if local LLM inference is desired.

Optional:

AWS CLI.
kubectl.
Helm.
Terraform.
A local Kubernetes distribution.
17.2 Local services

Recommended local topology:

text
Vue UI:       http://localhost:5173
Spring API:   http://localhost:8080
PostgreSQL:   localhost:5432
Ollama:       localhost:11434


These are example development ports. Change them if they conflict with other local services.

17.3 Environment variables

Create .env from .env.example.

dotenv
APP_ENV=local
APP_PORT=8080

DB_HOST=localhost
DB_PORT=5432
DB_NAME=supplier_ai
DB_USERNAME=supplier_ai_app
DB_PASSWORD=change-me-local-only

LLM_PROVIDER=ollama
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=<your-supported-local-model>

EMBEDDING_PROVIDER=ollama
EMBEDDING_MODEL=<your-supported-embedding-model>

VECTOR_DIMENSION=<matching-embedding-dimension>

ANTHROPIC_API_KEY=
OPENAI_API_KEY=

ATLASSIAN_MCP_URL=
ATLASSIAN_AUTH_MODE=
ATLASSIAN_TOKEN=

INTERNAL_API_BASE_URL=
GRAPHQL_API_URL=

REDIS_HOST=localhost
REDIS_PORT=6379

HARNESS_PROFILE=development


These variables illustrate the configuration contract. The actual application must bind and validate them. Do not commit .env, API keys, real credentials, or production data.

The chat and embedding models may be different. The configured vector dimension must match the embedding output exactly.

17.4 Start PostgreSQL

For local development, use a PostgreSQL image with a compatible pgvector extension. The exact image tag should be pinned in the repository.

Illustrative Compose service:

yaml
services:
  postgres:
    image: pgvector/pgvector:<pinned-compatible-tag>
    environment:
      POSTGRES_DB: supplier_ai
      POSTGRES_USER: supplier_ai_app
      POSTGRES_PASSWORD: change-me-local-only
    ports:
      - "5432:5432"
    volumes:
      - supplier_pg_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U supplier_ai_app -d supplier_ai"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:
  supplier_pg_data:


The tag is intentionally a placeholder. Replace it with a real, pinned image version verified against the selected PostgreSQL and pgvector versions.

For a production-grade Compose setup, use separate migration and application database roles rather than granting schema ownership to the runtime account.

17.5 Start the local model

Install Ollama using its official installation instructions, then pull a compatible model:

bash
ollama pull <your-supported-local-chat-model>
ollama pull <your-supported-local-embedding-model>
ollama list


Use models appropriate to your available CPU, RAM, GPU, and quality requirements. The placeholders must be replaced with actual models available in your Ollama installation.

17.6 Build and run the backend

After implementing the Maven modules:

bash
mvn clean verify
mvn -pl apps/supplier-ai-api spring-boot:run


For a multimodule Maven project, configure the necessary parent modules and dependencies so that the API module can resolve the shared libraries. Add a separate command for the ingestion worker after it is implemented.

17.7 Run the frontend
bash
cd frontend/supplier-ai-ui
npm ci
npm run dev


Expected local frontend address: http://localhost:5173.

The Vue application should read the API base URL from a controlled configuration mechanism. Never place provider API keys in frontend environment variables exposed to browser code.

18. How to Run and Test
18.1 Recommended local startup sequence
Start PostgreSQL and verify the health check.
Apply database migrations.
Load synthetic supplier fixtures.
Start the local LLM and verify model availability.
Start the Spring Boot API.
Start the Vue frontend.
Run document ingestion on synthetic Word, PDF, and Excel files.
Configure the mock or test MCP connector.
Execute API and retrieval tests.
Run the evaluation dataset and inspect the trace output.
18.2 Suggested Makefile

The following targets are examples to implement in the repository:

makefile
.PHONY: build test integration-test ui-test eval

build:
	mvn clean package

test:
	mvn test

integration-test:
	mvn verify -Pintegration-tests

ui-test:
	cd frontend/supplier-ai-ui && npm ci && npm run test:unit

eval:
	mvn -pl apps/supplier-ai-evaluator spring-boot:run


Configure the Maven profiles and application arguments before using these targets. Avoid running production-connected tests as an accidental side effect of the default build.

18.3 API test examples

Assuming the implemented API exposes these endpoints:

bash
curl http://localhost:8080/actuator/health


Expected result: a successful health response when the application is ready.

Example conversational request:

bash
curl -X POST http://localhost:8080/api/v1/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <local-test-token>" \
  -d '{
    "conversationId": null,
    "message": "Compare supplier performance for the last quarter"
  }'


The endpoint and JSON schema are proposed contracts, not claims that an implementation already exists. Use an actual development identity provider or test authentication mechanism; do not disable production authentication to make this request work.

18.4 Integration test scenarios

Test at least the following:

Supplier master data can be retrieved through an approved domain service.
Risk history returns the correct time period and timestamps.
Performance calculations match known fixture values.
Documents are parsed and indexed with source locations.
Vector queries return relevant chunks.
Lexical search finds exact supplier codes and Jira issue keys.
Confluence and Jira access respects source permissions.
REST and GraphQL failures are handled safely.
MCP tools reject unauthorized and malformed inputs.
LLM output with fabricated citations is rejected or corrected.
Stale cached results are not returned beyond their allowed freshness window.
Prompt injection in a retrieved document cannot authorize a prohibited tool call.
Maximum iteration and tool-call limits are enforced.
Missing or conflicting evidence results in an appropriate qualification or abstention.
18.5 Definition of a successful local run

A local setup is ready for development when:

Migrations complete successfully.
Synthetic supplier records are queryable.
A sample document can be ingested and retrieved.
The configured model can generate an answer.
The answer contains valid source references.
Integration and security tests pass.
The evaluation report is generated and versioned.

Do not mark the platform production-ready solely because a sample question receives a plausible answer.

19. Kubernetes Deployment
19.1 Kubernetes architecture
mermaid
flowchart TB
    USER[Users]
    ING[Ingress or AWS ALB]
    UI[UI Deployment]
    API[Spring API Deployment]
    WORKER[Ingestion Worker Deployment]
    EVAL[Evaluation Jobs]
    DB[(Managed PostgreSQL with pgvector)]
    S3[(Object Storage)]
    SECRETS[External Secret Management]
    OBS[Metrics, Logs and Traces]

    USER --> ING
    ING --> UI
    ING --> API
    API --> DB
    API --> S3
    WORKER --> DB
    WORKER --> S3
    SECRETS --> API
    SECRETS --> WORKER
    API --> OBS
    WORKER --> OBS
    EVAL --> DB

19.2 Deployment workloads
Deployment: API pods.
Deployment: Vue UI pods or static hosting service.
Deployment: ingestion workers.
CronJob: scheduled synchronization and evaluations.
Job: database migration, controlled indexing, or batch reindexing.
Service: internal API routing.
Ingress or AWS Load Balancer Controller: external access.
HorizontalPodAutoscaler: stateless application scaling.
NetworkPolicy: restricted pod-to-pod communication where supported.

Use managed PostgreSQL for production unless there is a strong, documented reason to operate the database inside Kubernetes.

19.3 Example Kubernetes deployment

Illustrative deployment manifest:

yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: supplier-ai-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: supplier-ai-api
  template:
    metadata:
      labels:
        app: supplier-ai-api
    spec:
      serviceAccountName: supplier-ai-api
      containers:
        - name: api
          image: <account>.dkr.ecr.<region>.amazonaws.com/supplier-ai-api:<immutable-tag>
          ports:
            - name: http
              containerPort: 8080
          env:
            - name: APP_ENV
              value: production
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: supplier-ai-runtime
                  key: db-host
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: http
            initialDelaySeconds: 20
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: http
            initialDelaySeconds: 30
            periodSeconds: 15
          resources:
            requests:
              cpu: "500m"
              memory: "1Gi"
            limits:
              cpu: "2"
              memory: "3Gi"
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsNonRoot: true


The resource values are starting examples, not a sizing recommendation. Benchmark under expected load, configure a non-root image user, add writable temporary volumes only where needed, and configure all required environment variables and secret references.

For AWS, use IAM Roles for Service Accounts (IRSA) or the appropriate EKS Pod Identity configuration for AWS access. Do not place long-lived AWS credentials in Kubernetes Secrets when workload identity can be used.

19.4 Production deployment controls

Implement:

Rolling deployments and readiness gates.
Pod disruption budgets.
Resource requests and limits.
Autoscaling based on appropriate metrics.
Graceful shutdown and request draining.
Network restrictions and workload identity.
Secret rotation.
Image scanning and immutable tags.
Migration jobs with controlled execution.
Backup verification and disaster recovery tests.
Rollback procedures for application and harness versions.

A new embedding dimension or incompatible index schema may require a migration and reindexing plan, not simply a rolling deployment.

20. AWS Production Deployment
20.1 Recommended AWS reference architecture
AWS service	Purpose
Amazon EKS	Run Spring Boot APIs and background workers
Amazon ECR	Store container images
Amazon RDS for PostgreSQL	Managed structured and vector database, subject to extension and version compatibility
Amazon S3	Store original documents and ingestion artifacts
AWS Secrets Manager	Manage API keys and database credentials
AWS KMS	Encryption key management
Application Load Balancer	HTTPS ingress
AWS WAF	Web request filtering where appropriate
IAM and EKS workload identity	Least-privilege AWS access
Amazon CloudWatch	Logs, metrics, and operational alarms
AWS X-Ray or OpenTelemetry-compatible tracing backend	Distributed tracing
Amazon ElastiCache for Redis	Optional shared cache
AWS Backup	Managed backup policies where applicable
Terraform	Reproducible infrastructure provisioning

RDS PostgreSQL and pgvector are a reasonable default, but verify the supported PostgreSQL engine version, extension availability, parameter settings, performance requirements, and regional service constraints before provisioning.

20.2 Network design

Use a VPC spanning multiple Availability Zones.

Recommended separation:

Public subnets for the load balancer where required.
Private subnets for EKS worker nodes.
Private database subnets for RDS.
Restricted egress for model APIs and enterprise integrations.
VPC endpoints for supported AWS services where beneficial.
Explicit security groups and network policies.

Do not expose PostgreSQL or internal MCP endpoints publicly.

If an external LLM API is used, the workload needs a controlled outbound path to the provider. Review the provider's data handling, retention, regional processing, and contractual terms before transmitting supplier information.

20.3 Data storage design

RDS PostgreSQL

Supplier domain data and approved integrated ERP data.
Knowledge metadata and chunk storage.
pgvector indexes.
AI configuration and evaluation metadata.

S3

Original documents.
Temporary ingestion artifacts.
Evaluation reports.
Approved model artifacts.
Optional exports and archival records.

Use encryption, lifecycle policies, least-privilege bucket access, and retention rules. Avoid storing duplicate sensitive content without a business need.

20.4 Deployment sequence
Provision the VPC, subnets, routes, security groups, and required endpoints.
Provision RDS PostgreSQL and verify pgvector compatibility.
Configure encryption, backups, monitoring, and database access.
Provision EKS and workload identity.
Create ECR repositories and build/publish immutable images.
Configure Secrets Manager and workload permissions.
Deploy the application, ingestion worker, and UI.
Run database migrations as a controlled deployment job.
Configure ingress, TLS certificates, and WAF where appropriate.
Connect approved enterprise integrations.
Ingest a controlled subset of documents.
Run integration, security, and AI evaluation suites.
Validate dashboards, alarms, backup restoration, and rollback.
Gradually enable production users.
20.5 Infrastructure as code

Keep AWS infrastructure in version-controlled Terraform modules.

Recommended modules:

text
deploy/terraform/
├── modules/
│   ├── network/
│   ├── eks/
│   ├── rds-postgresql/
│   ├── s3-documents/
│   ├── secrets/
│   ├── observability/
│   └── ingress/
└── environments/
    ├── dev/
    ├── staging/
    └── prod/


Use remote Terraform state with encryption, locking supported by the selected backend, reviewed plans, and restricted state access. Never commit state files containing sensitive values.

20.6 Production availability and recovery

Define recovery requirements before choosing database topology.

Document:

Recovery Point Objective (RPO).
Recovery Time Objective (RTO).
Database backup retention.
Point-in-time recovery procedures.
Cross-region recovery requirements, if applicable.
Document-store recovery.
Reindexing time for the vector corpus.
Provider outage behavior.
Degraded-mode behavior when Jira or Confluence is unavailable.

A recoverable database backup does not automatically mean the entire AI system is recoverable. Model configurations, prompt versions, embedding versions, source connectors, and evaluation datasets must also be preserved.

20.7 Cost management

Track cost by workload and model provider.

Major cost drivers include:

LLM input and output tokens.
Embedding generation and reindexing.
RDS instance size, storage, and I/O.
EKS compute and networking.
S3 storage and requests.
Redis memory.
Logging and trace volume.
OCR or other document processing.

Set per-request and per-tenant budgets where applicable. Evaluate the cost of reindexing and model changes before rollout.

21. Observability and Operations
21.1 Required telemetry

Every request should have a correlation ID and, where appropriate, a trace ID connecting API processing, retrieval, SQL calls, MCP calls, model invocations, and output validation.

Measure:

Request volume and error rate.
End-to-end latency.
Retrieval latency and result counts.
SQL and vector query latency.
Model-provider latency and errors.
Token usage and estimated cost.
Cache hit rates.
Ingestion lag and failed documents.
Index freshness.
Citation validation failures.
Guardrail blocks.
Tool-call failures.
Evaluation regressions.
User feedback.
21.2 Example operational objectives

Set service-level objectives based on measured demand and business impact. For example:

API availability target.
p95 latency for common supplier queries.
Maximum acceptable document indexing lag.
Maximum age for designated high-risk data.
Error budget for external model-provider failures.
Minimum retrieval and grounding quality.

Do not apply a single latency objective to all requests. A simple supplier lookup and a multi-source analysis have different execution profiles.

21.3 Incident runbooks

Create runbooks for:

LLM provider outage.
PostgreSQL connection exhaustion.
Vector search degradation.
Failed or delayed ingestion.
Atlassian MCP outage.
Incorrect or stale supplier information.
Unauthorized data access.
Prompt-injection incident.
Unexpected cost spike.
Failed model or embedding rollout.

Operational procedures should define detection, containment, rollback, escalation, and recovery verification.

22. CI/CD and MLOps
22.1 Continuous integration

For every pull request:

Compile Java modules.
Run unit tests.
Validate frontend types and lint rules.
Run database migration tests.
Run integration tests against isolated dependencies.
Run static analysis and dependency scanning.
Validate configuration schemas.
Run a fast retrieval and answer regression suite.
Build immutable container images.
Publish test and security reports.
22.2 Continuous delivery

Recommended release flow:

text
Pull request
    |
    v
Build and test
    |
    v
Security and quality checks
    |
    v
Publish immutable artifacts
    |
    v
Deploy to development
    |
    v
Integration and AI evaluation
    |
    v
Promote to staging
    |
    v
Security and acceptance approval
    |
    v
Production canary
    |
    v
Monitor and promote or rollback

22.3 Versioned AI artifacts

Maintain version identifiers for:

Harness profile.
System and task prompts.
Model provider and model ID.
Embedding model.
Reranker.
Chunking profile.
Retrieval weights.
Evaluation dataset.
Fine-tuned model or adapter.
Source index version.

Each evaluation result should record the versions it tested. Without this information, a production quality regression is difficult to reproduce.

22.4 Release gates

Block promotion if:

Critical security tests fail.
Tenant isolation fails.
Citation validation regresses below the approved threshold.
Required source connectors fail contract tests.
A new model causes a material regression in the golden evaluation set.
A migration is not backward-compatible with the rollout plan.
Cost or latency exceeds an approved limit without an explicit exception.
23. Implementation Roadmap

A production implementation should be delivered in controlled increments rather than attempting every advanced feature at once.

Phase 1 — Foundation
Create the Maven monorepo.
Establish Spring Boot API conventions.
Implement authentication and authorization.
Provision PostgreSQL and Flyway migrations.
Implement supplier domain services.
Create the Vue conversational interface.
Add synthetic test data and CI.

Exit criteria: A secure application can retrieve and display supplier facts without an LLM.

Phase 2 — Core RAG
Implement document ingestion.
Add Word, PDF, and Excel parsers.
Add pgvector storage and retrieval.
Add embedding-provider adapters.
Implement chunking profiles.
Add source metadata and citations.
Implement basic query routing and response synthesis.

Exit criteria: Document questions return grounded, cited responses.

Phase 3 — Enterprise integrations
Integrate Atlassian MCP.
Implement the custom MCP server.
Add REST and GraphQL connectors.
Implement incremental synchronization.
Add source permission mapping and audit logs.

Exit criteria: Authorized users can retrieve evidence from approved enterprise sources.

Phase 4 — Configurable harness
Implement model routing.
Add query planning and bounded execution loops.
Add retrieval and prompt configuration.
Implement retries, deadlines, caching, and fallbacks.
Add configuration validation and versioning.

Exit criteria: Harness profiles can be changed and evaluated without rewriting the core orchestration logic.

Phase 5 — Explainability and evaluation
Implement the evidence ledger.
Add citation validation.
Create golden evaluation datasets.
Add retrieval and answer quality metrics.
Add feedback collection and regression gates.
Test adversarial documents and access-control boundaries.

Exit criteria: Answer quality and safety can be measured, reproduced, and audited.

Phase 6 — Advanced ML capabilities
Compare embedding models.
Introduce reranking.
Add more sophisticated chunking experiments.
Build optional fine-tuning pipelines.
Add controlled model and profile promotion.

Exit criteria: Changes to model or retrieval quality are evidence-based and reversible.

Phase 7 — Production deployment
Deploy to AWS EKS.
Configure managed PostgreSQL, S3, secrets, and monitoring.
Add CI/CD and infrastructure as code.
Run load, security, and disaster recovery tests.
Complete operational runbooks and access reviews.

Exit criteria: The platform meets agreed availability, security, performance, recovery, and AI quality objectives.

24. Definition of Done

A production release is ready only when the relevant requirements below are satisfied.

Business and data
Supplier, sourcing, risk, performance, ERP, and supply-chain queries use authoritative sources.
Business calculations match verified expected results.
Data freshness is visible where it matters.
Conflicting source data is handled explicitly.
Word, PDF, Excel, Confluence, and Jira ingestion has been validated.
RAG and AI
Hybrid retrieval is implemented and evaluated.
Chunking and embedding configurations are versioned.
The model router supports approved providers.
Tool calls and agent loops are bounded.
Citations are validated against retrieved evidence.
Unsupported questions result in an appropriate clarification or abstention.
Evaluation results are reproducible.
Security
Authentication and authorization are enforced.
Tenant and document ACL isolation has passed adversarial tests.
No unrestricted production SQL or GraphQL execution is available to the LLM.
Secrets are not committed to source control.
Prompt injection cannot grant additional privileges.
Sensitive actions require the appropriate approval.
Logging and retention policies have been reviewed.
Operations
Database migrations and rollback plans are documented.
Dashboards and alerts are configured.
Backup restoration has been tested.
Model-provider outage behavior is tested.
Container images are scanned and pinned.
AI configuration and model changes are versioned.
Recovery objectives and operational ownership are documented.
25. References

Use official documentation to validate API details, version compatibility, licensing, and deployment behavior during implementation.

Core technologies
Java: https://dev.java/
Spring Boot: https://spring.io/projects/spring-boot
Spring AI: https://spring.io/projects/spring-ai
PostgreSQL: https://www.postgresql.org/docs/
pgvector: https://github.com/pgvector/pgvector
Vue.js: https://vuejs.org/
TypeScript: https://www.typescriptlang.org/docs/
Apache Tika: https://tika.apache.org/
Apache POI: https://poi.apache.org/
AI and integrations
Ollama: https://ollama.com/
OpenAI API: https://platform.openai.com/docs/
Anthropic API: https://docs.anthropic.com/
Model Context Protocol: https://modelcontextprotocol.io/
Atlassian developer documentation: https://developer.atlassian.com/
Deployment and observability
Docker: https://docs.docker.com/
Kubernetes: https://kubernetes.io/docs/
Amazon EKS: https://docs.aws.amazon.com/eks/
Amazon RDS for PostgreSQL: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html
Amazon S3: https://docs.aws.amazon.com/s3/
Terraform: https://developer.hashicorp.com/terraform/docs
OpenTelemetry: https://opentelemetry.io/docs/
Final Architectural Recommendation

Build the first production-capable version around Spring Boot, PostgreSQL/pgvector, a provider-neutral RAG harness, and a Vue.js conversational UI. Prioritize correct structured-data access, permission-aware retrieval, source citations, and evaluation before adding complex autonomous agent behavior.

Keep fine-tuning, sophisticated dynamic chunking, reranking experiments, and advanced multi-agent workflows behind configurable interfaces. This makes the system extensible without turning the initial implementation into an unnecessarily complex platform.

The defining quality of this solution should be that procurement and supply-chain teams can not only receive an answer, but also verify the evidence, understand its limitations, trace how the answer was produced, and trust that the system respected their permissions.
