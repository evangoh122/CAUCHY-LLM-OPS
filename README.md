A stage-by-stage evaluation framework for an AML/KYC agent built on Databricks. Every pipeline stage is traced and scored in MLflow, and the results are displayed as a six-sided Minesweeper cube: each face is a stage, each tile is a test case or assertion, and one failed critical assertion loses the stage.

The goal is to find where the system fails, explain why, and tell a genuine compliance failure apart from an appropriate escalation to a human.

How it works

Each synthetic case runs through six stages. A stage has to pass before the case moves on.

#	Stage	What is traced	What is evaluated
1	Ingestion	Document ID, version, chunks, effective dates	Source and metadata correctness
2	Retrieval	Query, ranked clause IDs, scores	Recall of critical clauses
3	Applicability	Jurisdiction, entity, activity, dates	Rule selection
4	Reasoning (rules, graph, LLM)	Rule execution, graph edges, exceptions, prompt version, evidence references, response	Deterministic consistency, grounding, correctness of the conclusion
5	Decision	Final classification, escalation trigger	False clearances and escalation recall
6	Evaluation	Comparison against expected outcomes	Pass or fail per assertion

Every record keeps the trace_id, span_id, case_id, stage result, scorer version and failure reason. Custom spans capture intermediate steps, and custom scorers attach the assessments.

Tile states
State	Meaning
Passed	All mandatory evaluation gates satisfied
Failed	A critical assertion failed and the stage is lost
Escalated	The case correctly requires human judgment (counts as a pass)
Unevaluated	The test has not completed or there is not enough evidence

A correct escalation is a successful outcome. If two obligations conflict, the expected behaviour is to flag the conflict and escalate, not to invent a resolution.

MLflow is the source of truth

The cube only renders results recorded in MLflow. It never decides whether a regulatory result is right or wrong. Selecting a failed tile opens the trace with the expected behaviour, the actual result and the failed assertion.

Data
Layer	Source
Entities and events	Synthetic, generated with Faker and SDV: fictional customers, banks, ownership chains, transactions and onboarding cases
Regulation	Real, versioned sources: MAS, HKMA, DFSA, FSRA, UK, OFAC, Wolfsberg guidance
Expected outcomes	Set by the author, independently of the model under test

Any rule altered for stress testing is labelled as a simulated policy. Nothing fictional is presented as real law.

Case schema
json
{
  "case_id": "KYC-SYN-0042",
  "dataset_version": "v1",
  "jurisdiction": ["SG", "HK"],
  "entity_type": "corporate",
  "business_activity": "customer_onboarding",
  "as_of_date": "2026-11-15",
  "source_obligation_ids": [],
  "expected_applicable_rules": [],
  "expected_exceptions": [],
  "expected_evidence_ids": [],
  "expected_decision": "ESCALATE",
  "expected_escalation": true,
  "expected_failure_stage": null,
  "synthetic": true
}
Tech stack
Databricks: [TODO: which features, e.g. Delta tables, Unity Catalog, Asset Bundles, Vector Search]
MLflow: tracing, custom scorers, evaluation runs
Faker and SDV: synthetic entity and event generation
uv: Python environment and dependency management
Rules engine: [TODO]
Knowledge graph: [TODO]
LLM: [TODO]
Minesweeper cube: [TODO: frontend framework]
Repository structure
aml-kyc-eval/
├── pyproject.toml      # project metadata and dependencies
├── uv.lock             # locked dependency versions
├── .python-version
├── README.md
└── src/
    └── aml_kyc_eval/
        └── __init__.py
[TODO: pipeline, scorers, synthetic data and cube folders]
Getting started
Prerequisites
Python 3.11 or later
uv
[TODO: Databricks workspace access and credentials]
Setup
bash
git clone [TODO: repo URL]
cd aml-kyc-eval
uv sync

uv sync creates the virtual environment and installs the locked dependencies from uv.lock. To add a package, use uv add <package>.

Run the evaluation
bash
[TODO]
Launch the cube
bash
[TODO]
Modes

Replay mode (current). MLflow has already evaluated each case. Revealing a tile shows the recorded result, so every outcome is reproducible.

Live mode (planned). Revealing a tile runs the test against the Databricks pipeline. This needs an execution API and handling for cost, latency and repeatability.

Roadmap
[ ] Fault injection: remove a critical passage, corrupt an effective date, drop a graph edge, change an ownership percentage, then check that the right gate catches each defect
[ ] Live mode
Disclaimer

This is a personal project built in my own time. All customers, entities and transactions are synthetic. Views are my own and do not represent my employer.

License

[TODO]