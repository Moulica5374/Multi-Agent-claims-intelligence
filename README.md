# Multi-Agent Claims Intelligence System

A production-grade, multi-agent AI pipeline for automated insurance claims triage built on AWS. The system coordinates three specialized AI agents through a LangGraph state machine to process claims in seconds versus days with manual processing.

## Architecture

```
Claim Input (JSON/PDF)
        │
        ▼
┌─────────────────┐
│   FastAPI       │  REST API layer
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   LangGraph     │  Agent orchestration
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
Agent 1    Agent 2
Extraction  Retrieval
    │         │
    └────┬────┘
         ▼
      Agent 3
      Decision
         │
         ▼
  Straight Through /
  Adjuster Review /
  Fraud Investigation
```

## Tech Stack

| Component | Technology |
|-----------|------------|
| Agent Framework | CrewAI |
| Orchestration | LangGraph |
| LLM | AWS Bedrock Nova |
| Embeddings | AWS Bedrock Titan Embed v2 |
| Vector Store | FAISS (ANN retrieval) |
| Document Ingestion | AWS Textract |
| Data Storage | AWS S3 |
| API Layer | FastAPI |
| Validation | Pydantic |
| Observability | LangSmith |
| Compute | AWS SageMaker |
| Evaluation | Ragas |

## Agents

### Agent 1 — Claims Extraction
- Accepts claims in JSON and PDF formats
- Extracts structured fields using Bedrock Nova LLM
- Validates against Pydantic schema
- Flags missing or ambiguous fields

### Agent 2 — Policy Retrieval
- Semantic search over 500K+ policy documents
- Two layer retrieval — FAISS ANN + metadata filter
- Returns relevant policy context with confidence score
- Filters by policy type, state, active status

### Agent 3 — Routing Decision
- Synthesizes claim details and policy context
- Makes routing decision with confidence scoring
- Reflection loop when confidence is low
- Human in the loop gate for unresolved cases

## RAG Pipeline

- 500K+ insurance policy documents ingested
- PDF → AWS Textract → JSON normalization
- Grouped chunking — 20-50 policies per chunk by type and state
- Bedrock Titan Embed v2 — 1024 dimension embeddings
- FAISS HNSW index for approximate nearest neighbor retrieval
- Ragas evaluation — Context Precision > 0.85, Faithfulness > 0.85

## Performance

| Metric | Value |
|--------|-------|
| Manual processing time | 2.5 days |
| AI processing time | 8 seconds |
| Straight through rate | 70% |
| Adjuster review rate | 25% |
| Fraud flag rate | 5% |
| RAG Context Precision | > 0.85 |
| RAG Faithfulness | > 0.85 |

## Project Structure

```
multi-agent-claims-intelligence/
├── notebooks/
│   ├── 01_data_generation.ipynb
│   ├── 02_rag_pipeline.ipynb
│   ├── 03_agent1_extraction.ipynb
│   ├── 04_agent2_retrieval.ipynb
│   ├── 05_agent3_decision.ipynb
│   └── 06_langgraph_orchestration.ipynb
├── src/
│   ├── agents/
│   │   ├── agent1_extraction.py
│   │   ├── agent2_retrieval.py
│   │   └── agent3_decision.py
│   ├── rag/
│   │   ├── chunking.py
│   │   ├── embeddings.py
│   │   └── faiss_index.py
│   ├── orchestration/
│   │   └── langgraph_pipeline.py
│   └── api/
│       └── fastapi_app.py
├── configs/
│   └── agents.yaml
├── evaluation/
│   └── ragas_eval.py
├── requirements.txt
└── README.md
```

## Setup

```bash
# Clone repo
git clone https://github.com/Moulica5374/multi-agent-claims-intelligence.git
cd multi-agent-claims-intelligence

# Install dependencies
pip install -r requirements.txt

# Configure AWS credentials
aws configure

# Run RAG pipeline
jupyter notebook notebooks/02_rag_pipeline.ipynb

# Start API
uvicorn src.api.fastapi_app:app --reload
```

## Evaluation

```bash
# Run Ragas evaluation
python evaluation/ragas_eval.py
```

## Key Design Decisions

- **Grouped chunking** — 20-50 policies per chunk reduces Bedrock API calls by 30x vs individual embedding
- **Two layer retrieval** — FAISS semantic search narrows to relevant group, metadata filter finds exact policy
- **Confidence scoring** — each agent handoff includes confidence score, triggers reflection loop below threshold
- **Human in the loop** — unresolved cases after reflection routed to human reviewer, not silently failed

## License

MIT