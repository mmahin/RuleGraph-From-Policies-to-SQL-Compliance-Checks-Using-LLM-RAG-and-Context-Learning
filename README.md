
---

# 🧠 RuleGraph

**From Policies to SQL Compliance Checks Using LLM, RAG, and Context Learning — in one notebook**

This repository contains a single Jupyter/Colab notebook that turns **unstructured policy documents** (e.g., PDFs) into **machine-executable rules** and **SQL compliance checks** using **LLMs**, **Graph-RAG**, and an optional **Neo4j** knowledge graph.

---

## What’s inside the notebook

* **PDF/Text ingestion & chunking**
* **LLM rule extraction → structured JSON**
* **(Optional) Neo4j**: encode rules as a knowledge graph
* **Graph-RAG retrieval** of relevant rules
* **Tabular-RAG**: read your DB schema for context
* **SQL generation** for compliance checks
* **Execution & reporting** (list violations, summarize results)

> The notebook is self-contained. You can run end-to-end without writing extra code.

---

## Quick start

### Option A — Run in Google Colab (recommended)

1. Upload the notebook to Colab.
2. (Optional) Mount Drive if your PDFs are in Drive.
3. Run **Runtime → Run all** and follow the on-screen prompts in config cells.

### Option B — Run locally (Jupyter/Lab)

```bash
# create and activate env
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

# minimal deps
pip install jupyter pandas tqdm PyMuPDF neo4j SQLAlchemy pymysql python-dotenv

# launch
jupyter lab  # or: jupyter notebook
```

---

## Minimal requirements

* **Python** 3.9+
* **Jupyter/Colab**
* For SQL checks: a reachable database (e.g., MySQL)
* For KG features (optional): a **Neo4j** instance (local Docker or Aura)

---

## Configure your run (in-notebook)

The first config cell exposes simple variables:

* `PDF_FOLDER`: path containing your policy PDFs (e.g., `./docs/` or a Drive path)
* `DB_URI`: e.g., `mysql+pymysql://user:pass@localhost:3306/finance`
* `USE_KG`: `True/False` (toggle Neo4j features)
* `NEO4J_URI`, `NEO4J_USER`, `NEO4J_PASSWORD` (if `USE_KG=True`)
* `MODEL_EXTRACT`, `MODEL_CODEGEN`: names/aliases of the LLMs you use
* `DEFAULT_TABLE`: table to validate (e.g., `transactions`)

> If you don’t have LLM API keys set, the notebook includes a **mock extractor** example so you can test the flow.

---

## Optional: start Neo4j locally (Docker)

If you want the KG steps:

```bash
docker run -it --rm \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/neo4jpassword \
  -e NEO4J_dbms_memory_heap_initial__size=1G \
  -e NEO4J_dbms_memory_heap_max__size=1G \
  --name rulegraph-neo4j neo4j:5.23
```

Then set in the notebook:

```
USE_KG = True
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASSWORD = "neo4jpassword"
```

---

## Data you provide

* **Policy PDFs**: Put them in a folder you reference via `PDF_FOLDER`.
* **Database**: Ensure the target DB is reachable and contains the tables/columns your rules refer to (e.g., `transactions.amount`, `transaction_time`, `report_time`).
* **Schema**: The notebook will introspect your schema automatically (SQLAlchemy inspector).

---

## Typical run flow (by notebook sections)

1. **Setup & deps**: installs imports (Colab) or checks local env.
2. **Config**: set paths, DB URI, LLM names, Neo4j toggles.
3. **Ingest**: load PDFs, chunk text with overlap to preserve context.
4. **Extract rules (LLM)**: convert text to JSON rules (mock or real LLM).
5. **(Optional) KG encode**: upsert rules/entities/conditions into Neo4j.
6. **Graph-RAG**: query rules by entity/intent (e.g., “Transaction” rules).
7. **Tabular-RAG**: read DB schema; assemble a schema-aware prompt.
8. **SQL generation**: produce SQL for each rule (safe fallbacks if columns missing).
9. **Execute checks**: run SQL; list violating rows per rule.
10. **Report**: print a compact compliance summary.

---

## Outputs

* **Rule JSONs** (in memory; optional cell to save to `./output/rules.json`)
* **Neo4j graph** (if enabled)
* **Generated SQL** for each rule
* **Compliance report** (violations per rule; optional CSV export)

---

## Troubleshooting

* **No rules extracted?**
  Start with the **mock extractor** cell enabled and/or test with a known phrase like “**$10,000**” to trigger the example rule.
* **SQL errors / missing columns**
  The generator tries to match columns. If a column isn’t found, it falls back to safe defaults—adjust the cell `DEFAULT_TABLE` or conditions.
* **Neo4j connection refused**
  Verify container is running, creds match, and the Bolt port `7687` is open.
* **DB connection fails**
  Confirm `DB_URI`, driver installed (e.g., `pymysql`), and user privileges.

---

## Roadmap (notebook)

* Add multi-hop Graph-RAG (jurisdiction → rule → thresholds)
* Optional **Streamlit** cell to render a quick dashboard
* Rule confidence scoring & learning from audit outcomes
* Multi-domain templates (finance, healthcare, environmental)

---

## License

**MIT** — free to use, modify, and share.

---

## Citation (optional)

If this notebook helps your work, consider citing the repo in presentations or internal docs as:

> Md Mahin, “RuleGraph (Notebook): From Policies to SQL Compliance Checks Using LLM, RAG, and Context Learning,” 2025.

---

### One-liner for GitHub “About”

> *Notebook that converts policy PDFs to executable SQL checks via LLM extraction, Graph-RAG, and optional Neo4j.*
