# RTGD
Carbon governance is moving from reporting to real‑time control. With CBAM, China’s market, and ISSB S2, carbon data now drives daily operations. Yet siloed EMS, MES, and trading systems erode quality. RTGD makes quality intrinsic—each record includes metadata on freshness, validity, and traceability, ensuring decision‑ready, trustworthy data.
RTGD-Orchestrator is not yet another carbon management software product—it is a paradigm proposal for carbon data infrastructure. Its core position is that carbon data quality problems cannot be solved through after-the-fact cleansing; they must be addressed at the root by embedding quality awareness at the data's point of origin.

This blueprint describes a vision that unfolds progressively. Phase 0 proves the concept's viability with a minimum system. Phase 1 validates the engineering feasibility of multi-Agent collaboration within a single-park closed loop. Phase 2 extends the system to supply chain and international compliance scenarios. Phase 3 unlocks the financial value of carbon data. Each Phase delivers independent value—users need not wait for all features to be ready before they begin benefiting.

As an open-source project, RTGD-Orchestrator's success ultimately depends on whether it can build an active community ecosystem. ToolSpec's open interface design, the quality scoring engine's open rule registration mechanism, and the "protocol over implementation" philosophy are all aimed at lowering the participation barrier and encouraging diverse contributions. We expect RTGD's value to manifest not only within this single project but in the broader industry adoption of the carbon data quality standards and collaboration protocols it promotes.

The evolution of carbon data from "usable" to "useful" to "trustworthy" is the technical foundation for carbon governance's transition from rhetoric to results. RTGD-Orchestrator's mission is to provide an open, trusted, and evolvable technical foundation for this evolution.
┌─────────────────────────────────────────────────────────────────┐
│ Supply Chain Layer (Surface) │
│ Supply Chain Agent · Product CFP Tracking │
├─────────────────────────────────────────────────────────────────┤
│ Park Platform Layer (Line) │
│ Park Agent · Carbon Budget · Demand Response · Optimization │
├──────────┬──────────┬──────────┬──────────┬─────────────────────┤
│ Factory │ Factory │ Factory │ Factory │ │
│ Agent A │ Agent B │ Agent C │ Agent D │ ... (Point Layer) │
├──────────┴──────────┴──────────┴──────────┴─────────────────────┤
│ RTGD Core Engine │
│ Emission Factors · Audit Chain · Event Bus · Provenance Engine │
├─────────────────────────────────────────────────────────────────┤
│ Integration & Assetization Layer │
│ ToolSpec · Four-Party Interface · Data Assetization · Vault │
└─────────────────────────────────────────────────────────────────┘


**Trust boundaries are explicit.** Factory A cannot see Factory B's data. Cross-factory aggregation happens only at the Park tier. All external data is quality-scored before being trusted. Every cross-boundary operation is signed, authorized, and audit-logged.

> 📖 For the full architecture deep-dive, see [docs/architecture/overview.md](docs/architecture/overview.md).

## 🚀 Quick Start

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) 24.0+
- [Docker Compose](https://docs.docker.com/compose/install/) v2.20+

### Launch in 60 Seconds

```bash
# Clone the repository
git clone https://github.com/rtgd-orchestrator/rtgd-orchestrator.git
cd rtgd-orchestrator

# Start the single-factory demo (all components in one Docker Compose)
docker compose -f deploy/docker/single-factory.yml up -d

# Open the dashboard
open http://localhost:3000

This launches a complete single-factory environment with:

A Factory Agent collecting simulated equipment data via the Mock ToolSpec adapter
The RTGD Core Engine (embedded SQLite audit chain + NATS event bus)
The Quality Scoring Engine assessing every data point in real time
A Web Dashboard showing live carbon emission streams with quality scores
Explore with the CLI
bash
# Install the CLI
cargo install rtgd-cli

# Query real-time carbon emission data
rtgd data list --factory factory-demo --last 1h

# Inspect a specific data point's quality breakdown
rtgd data inspect <data-point-id>

# Trace a data point's full provenance chain
rtgd audit trace <data-point-id>

# Check system health
rtgd health

Your First ToolSpec Adapter
bash
# Scaffold a new adapter for your energy management system
rtgd scaffold adapter --name my-ems --protocol rest-api

# This generates:
#   packages/toolspec/adapters/my-ems/
#   ├── src/
#   │   ├── adapter.rs        # Implement describe(), execute(), health()
#   │   └── protocol.rs       # Protocol translation logic
#   ├── tests/
#   │   └── integration.rs    # Pre-built test harness
#   └── Cargo.toml

Fill in the protocol translation logic, run the tests, and your adapter is ready to plug in.

📦 Project Structure
mipsasm
rtgd-orchestrator/
├── packages/
│   ├── core/                     # RTGD Core Engine
│   │   ├── factor-service/       #   Versioned Emission Factor Service
│   │   ├── audit-chain/          #   Immutable Audit Hash Chain
│   │   ├── event-bus/            #   NATS-based Event Bus
│   │   └── provenance-engine/    #   Data Provenance Tracking
│   ├── quality-engine/           # Five-Dimensional Quality Scoring
│   │   ├── dimensions/           #   Scoring implementations
│   │   ├── rules/                #   Validation rule repository
│   │   └── registry/             #   Rule registration API
│   ├── agents/
│   │   ├── supply-chain-agent/   # Supply Chain Agent
│   │   ├── park-agent/           # Park Agent
│   │   └── factory-agent/        # Factory Agent
│   ├── toolspec/
│   │   ├── core/                 # ToolSpec interface definitions
│   │   ├── adapters/             # Built-in adapters (MyEMS, Modbus, REST, Mock)
│   │   └── scaffold-cli/        # Adapter scaffolding tool
│   ├── four-party/               # Four-Party Collaboration Connectors
│   ├── data-asset/               # Data Assetization Module
│   └── shared/                   # Shared types, protocols, crypto utils
├── apps/
│   ├── dashboard/                # Web monitoring dashboard
│   ├── cli/                      # Command-line tool
│   └── playground/               # Developer sandbox
├── docs/                         # Documentation
├── examples/                     # Deployment examples
├── deploy/                       # Docker / K8s / Edge configs
└── tests/                        # Unit / Integration / E2E tests

📐 Core Data Model
Every piece of carbon data in RTGD-Orchestrator flows as an RTGDDataPoint — the system's atomic data unit where value, quality, and provenance are structurally inseparable:

typescript
interface RTGDDataPoint {
  // Identification
  id: string;                     // UUIDv7 (time-ordered)
  source_id: string;              // Device, Agent, or system ID
  data_type: DataType;            // e.g., "electricity_consumption", "carbon_emission_direct"
  category: 'emission' | 'energy' | 'production' | 'financial' | 'factor' | 'derived';

  // Core value
  value: number;
  unit: string;                   // QUDT ontology
  uncertainty: {
    type: 'absolute' | 'relative';
    value: number;
    confidence_level: number;
  };

  // Timestamps
  measured_at: string;            // When it was actually measured
  reported_at: string;            // When it entered the system

  // Quality (INSEPARABLE)
  quality: QualityAssessment;     // Five-dimensional scoring

  // Provenance (INSEPARABLE)
  provenance: ProvenanceChain;    // Full origin-to-current trail

  // Context
  context: {
    facility_id: string;
    process_id?: string;
    product_batch_id?: string;
    emission_factor_id?: string;
    tags: Record<string, string>;
  };

  // Security
  signature: string;
  encryption_status: 'plaintext' | 'encrypted' | 'vault_sealed';
}

Quality Thresholds
Tier	quality_score	Use Case
Compliance Reporting	≥ 0.7	Regulatory submissions, CBAM, ISSB S2
Operational Decisions	≥ 0.5	Carbon budget management, demand response
Monitoring & Alerting	≥ 0.3	Dashboards, trend analysis, anomaly detection
Untrusted	< 0.3	Excluded from calculations; retained for diagnostics
📖 Full data model reference: docs/protocols/rtgd-datapoint.md

🔌 ToolSpec Ecosystem
ToolSpec is the universal adapter layer. Every external system — from a Modbus sensor to a cloud API — is wrapped behind three methods:

Method	Purpose
describe()	Returns structured capability description, input/output schemas, side-effect declarations
execute(params)	Invokes the tool; returns data + quality_score + provenance
health()	Returns real-time health: healthy / degraded / offline with diagnostics
Built-in Adapters
Adapter	Protocol	Status
Mock	Simulated data	✅ Available
MyEMS	REST API	✅ Available
Modbus-TCP	Modbus TCP/IP	✅ Available
REST Generic	Any REST API	✅ Available
OPC-UA	OPC Unified Architecture	🚧 In Progress
MQTT	MQTT 5.0	📋 Planned
Build your own in minutes with the scaffolding CLI:

bash
rtgd scaffold adapter --name my-device --protocol modbus

📖 ToolSpec specification: docs/protocols/toolspec.md

🌐 Four-Party Collaboration
Connector	Interfaces With	Capabilities
Technical	MEE (CN), Ember, IEA	Auto-sync emission factors with versioning
Regulatory	Compliance authorities	WebSocket real-time data subscription for continuous verification
Market	Carbon exchanges, CCER	Real-time carbon price feeds, order placement
International	ISSB, CSRD, CBAM	Semantic translation with auditable Translation Audit Reports
The International Translation Layer doesn't just convert formats — it documents every semantic mapping, information loss, assumption, and uncertainty delta in a machine-readable audit report.

🚢 Deployment Topologies
Single Machine (Trial / Single Factory)
bash
docker compose -f deploy/docker/single-factory.yml up -d

All components on one host. Minimum requirements: 4 CPU cores, 8 GB RAM, 50 GB SSD. Up and running in 5 minutes.

Park Deployment (5–50 Factories)
Park Cluster (3 nodes)              Edge Nodes (per factory)
┌─────────────────────┐            ┌──────────────────┐
│  Park Agent          │◄──mTLS───►│  Factory Agent    │
│  RTGD Core (PG+NATS)│            │  NATS Leaf Node   │
│  Quality Engine      │            │  Local Buffer     │
│  Four-Party Iface    │            └──────────────────┘
│  Dashboard           │                    × N
└─────────────────────┘

Factory edge nodes support store-and-forward during network outages. Reconnection triggers automatic state reconciliation with 100% data integrity recovery.

Enterprise Group (Multi-Park)
Multiple park clusters coordinated by a Supply Chain Agent at the enterprise level. Each park is an independent failure domain — one park's outage does not affect others.

📖 Deployment guides: deploy/

🔒 Security
RTGD-Orchestrator is designed for environments where carbon data is commercially sensitive and regulatorily consequential.

Identity & Authentication — Every Agent and user holds a unique key pair. Inter-Agent communication uses mutual TLS (mTLS). Human users authenticate via OpenID Connect integration with your existing identity provider.

Authorization — Attribute-Based Access Control (ABAC) powered by Open Policy Agent (OPA). Policies are version-controlled and all changes are audit-logged. Fine-grained rules like "Factory A's Agent may read only its own data from the Park Agent, only during business hours."

Data Protection — Three-layer encryption: TLS 1.3 in transit, Transparent Data Encryption at rest, and application-layer envelope encryption in the Data Vault. The enterprise holds the master key — not even the platform operator can decrypt the data.

Data Vault — Supports zero-knowledge-style verification: a financial institution can verify "are this enterprise's annual emissions below threshold X?" without seeing the actual figures.

📖 Security architecture: docs/architecture/security.md

🗺️ Roadmap
Phase 0 — Proof of Concept (Months 1–3)
 RTGDDataPoint data model and type libraries
 RTGD Core: emission factor service (built-in repo) + audit chain (SQLite)
 Quality engine: 3-dimension scoring (completeness, timeliness, consistency)
 Factory Agent: basic data collection + carbon calculation
 ToolSpec core interface + Mock adapter
 Single-machine Docker Compose deployment
Phase 1 — Single-Park Closed Loop (Months 4–8)
 Event bus upgrade to NATS JetStream
 Audit chain migration to PostgreSQL (HA)
 Quality engine: full 5-dimension scoring + rule registration API
 Park Agent: carbon budget manager + demand response engine
 ToolSpec: MyEMS adapter + Modbus-TCP adapter + scaffold CLI
 Park deployment topology with edge nodes
Phase 2 — Supply Chain & Four-Party (Months 9–14)
 Supply Chain Agent: single-tier → multi-tier carbon footprint tracking
 Technical connector (Ember, MEE) + Regulatory connector (WebSocket)
 Market connector (carbon exchange APIs) + International translator (MRV → ISSB S2)
 Automated CBAM compliance report generation
 Translation Audit Report framework
Phase 3 — Data Assetization & AI (Months 15–20)
 Carbon Asset Ledger + real-time valuation engine
 Authorization Loop Manager + Data Vault with zero-knowledge verification
 RL-based demand response optimization
 ML-based anomaly detection in quality scoring
 Predictive maintenance for Factory Agent
Phase 4 — Industry Ecosystem (Month 21+)
 Multi-industry template libraries (power, steel, cement, chemicals)
 Cross-enterprise carbon data interconnection protocol
 Standards track submissions (ISO TC207)
🤝 Contributing
We welcome contributions of all kinds! The fastest ways to get started:

🔌 Write a ToolSpec Adapter — Wrap a new device or system behind the three standard methods. No deep architecture knowledge needed. Use rtgd scaffold adapter to generate the boilerplate.

📏 Submit Quality Scoring Rules — Industry experts can contribute validation rules for specific sectors or equipment types in YAML format. No code required.

📝 Improve Documentation — Tutorials, translations, API examples, architecture diagrams.

🐛 Fix a Bug — Check issues labeled good first issue for curated starter tasks with context and estimated effort.

💡 Propose a Feature — Major changes go through our RFC process. Submit an RFC to start the conversation.

bash
# Set up the development environment
git clone https://github.com/rtgd-orchestrator/rtgd-orchestrator.git
cd rtgd-orchestrator

# Install dependencies
cargo build
pnpm install

# Run the test suite
cargo test --workspace
pnpm test

# Start the dev environment with hot reload
docker compose -f deploy/docker/dev.yml up -d

📖 Full contributor guide: CONTRIBUTING.md

📚 Documentation
Document	Description
Architecture Overview	System design, trust boundaries, communication model
RTGDDataPoint Specification	Core data model and quality framework
RTGDEvent Protocol	Inter-Agent communication protocol
ToolSpec Specification	Universal tool adapter interface
Security Architecture	Identity, authorization, encryption, Data Vault
Deployment Guide	Single-machine, park, and enterprise topologies
Your First Adapter	Step-by-step ToolSpec adapter tutorial
API Reference	Full API documentation
RFCs	Design proposals and decisions
🧭 Design Principles
Data Quality as Intrinsic Property — quality_score is not optional. Data without it is rejected.
Provenance as Audit — Every data point carries its full origin-to-current trail as a structural component.
Trust Domain Isolation — Factory A cannot see Factory B. External data is scored before being trusted.
Intent-Driven Layered Control — Upper tiers express what, lower tiers decide how.
Protocol Over Implementation — Conform to the protocol, swap the implementation.
Observability First — Every decision, every data hop, every tool invocation is observable and explainable.
Incremental Adoption — Start with one factory. Expand when ready. Every tier is self-contained.
📊 Performance Targets
Metric	Target
End-to-end data flow latency (sensor → Park Agent)	< 5 seconds
Quality scoring throughput	10,000+ data points/sec
Audit chain write latency	< 10 ms/entry
Data integrity recovery after 30-min disconnection	100%
Factory Agent minimum hardware	2 cores, 4 GB RAM (ARM/x86)
🏗️ Technology Stack
Component	Technology	Rationale
Core runtime	Rust	Memory safety, zero-cost abstractions, edge-to-cloud portability
Rapid prototyping	TypeScript	Connectors, dashboard, developer tools
Event bus	NATS JetStream	Lightweight, persistent streams, leaf-node hierarchy
Audit chain storage	SQLite (single) / PostgreSQL (cluster)	Append-only, no blockchain overhead
Time-series data	TimescaleDB	Shares engine with PostgreSQL, reduces ops burden
Service discovery	etcd	K8s-native integration
Authorization engine	OPA (Rego)	Fine-grained ABAC policies
📄 License
The core codebase is licensed under the Apache License 2.0.

Protocol specifications (RTGDDataPoint, RTGDEvent, ToolSpec) are published under CC BY 4.0, enabling anyone to freely implement these standards.

🌱 Community
Discord — Join our server for real-time discussion
GitHub Discussions — Ask questions, share ideas
Monthly Community Call — First Thursday of each month, 09:00 UTC. Calendar link
Twitter/X — @rtgd_orch for announcements
<div align="center">

The evolution from "usable" to "useful" to "trustworthy" carbon data is the technical foundation for carbon governance's transition from rhetoric to results.

RTGD-Orchestrator provides an open, trusted, and evolvable foundation for that evolution.
