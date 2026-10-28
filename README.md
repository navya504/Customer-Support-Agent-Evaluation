# Customer Support Agent Evaluation

This project evaluates an e-commerce customer support routing agent. Given a support ticket, the agent assigns one of five categories:

- `order_status`
- `refund_request`
- `product_issue`
- `account_help`
- `other`

The repository includes an executed evaluation notebook, the human-review workbook, exported results, and a short guide to the OpenTelemetry integration.

## Repository Contents

| File | Description |
| --- | --- |
| [`week4_customer_support_evals.ipynb`](week4_customer_support_evals.ipynb) | Executed notebook containing the classifier, evaluation workflow, prompt revision, results, and trace instrumentation. |
| [`llm_as_judge_jev.ipynb`](llm_as_judge_jev.ipynb) | Optional Section 11 notebook that compares a Qwen LLM-only judge with a Jev-first, 70%-confidence-gated judge and LLM fallback. |
| [`Week 4_ AI Evals (E-Commerce Customer Support Agent).xlsx`](Week%204_%20AI%20Evals%20%28E-Commerce%20Customer%20Support%20Agent%29.xlsx) | Evaluation workbook for reviewing failures, grouping error patterns, revising prompts, and optionally aligning an LLM judge. |
| [`results_v1.csv`](results_v1.csv) | Baseline predictions and evaluation results. |
| [`results_v2.csv`](results_v2.csv) | Improved-prompt predictions and evaluation results. |
| [`docs/opentelemetry_integration_one_pager.md`](docs/opentelemetry_integration_one_pager.md) | Overview of the OpenTelemetry and LangSmith tracing setup. |

## Results

| Run | Accuracy | Correct predictions |
| --- | ---: | ---: |
| Baseline prompt | 92% | 92 / 100 |
| Improved prompt | 97% | 97 / 100 |
| Change | +5 percentage points | +5 |

The revised prompt improved routing accuracy while preserving the individual prediction records needed for failure analysis.

## Evaluation Workflow

The notebook and workbook support the following review process:

1. Run the baseline classifier and compare predictions with the expected categories.
2. Annotate incorrect predictions and group recurring failure patterns.
3. Revise the classifier prompt to address a selected failure pattern.
4. Rerun the evaluation and compare the new results with the baseline.
5. Optionally test and align an LLM-as-judge evaluator.

## OpenTelemetry and LangSmith

Each ticket classification is wrapped in an OpenTelemetry span. The span records the ticket ID, prompt version, expected category, predicted category, correctness, and model reasoning. LangSmith receives these traces so individual failures can be inspected alongside the aggregate evaluation results.

See [`docs/opentelemetry_integration_one_pager.md`](docs/opentelemetry_integration_one_pager.md) for implementation details and troubleshooting guidance.

## Run the Notebook

Create a virtual environment and install the dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install jupyter langgraph "langsmith[otel]>=0.4.25" langchain-nebius langchain-core pandas scikit-learn pydantic tqdm opentelemetry-sdk opentelemetry-exporter-otlp
```

The notebook prompts for `NEBIUS_API_KEY` and `LANGSMITH_API_KEY` with hidden inputs if they are not already set. Enter the matching key at each prompt. Nebius Token Factory uses an OpenAI-compatible API through LangChain's `langchain-nebius` integration. The default endpoint is `https://api.tokenfactory.nebius.com/v1/` and the default model is `Qwen/Qwen3-30B-A3B-Instruct-2507`; override them with `NEBIUS_BASE_URL` and `NEBIUS_MODEL` if needed. LangSmith defaults to the US endpoint; set `LANGSMITH_ENDPOINT` for your region if your workspace is hosted elsewhere.

```bash
export NEBIUS_BASE_URL=https://api.tokenfactory.nebius.com/v1/
export NEBIUS_MODEL=Qwen/Qwen3-30B-A3B-Instruct-2507
export LANGSMITH_PROJECT=customer-support-evals
# Optional: use the API endpoint for your LangSmith region.
# export LANGSMITH_ENDPOINT=https://eu.api.smith.langchain.com
```

In PowerShell, set these values as `$env:NEBIUS_BASE_URL`, `$env:NEBIUS_MODEL`, and `$env:LANGSMITH_PROJECT`. Set `$env:LANGSMITH_ENDPOINT` to your regional LangSmith API URL when needed. The API keys can be entered at the notebook's hidden prompts.

Then launch the notebook:

```bash
jupyter notebook week4_customer_support_evals.ipynb
```

## Run the LLM-as-a-Judge notebook

After all 29 classifier failures have Step 3 labels, open `llm_as_judge_jev.ipynb`, choose one exact failure category, enter its rubric, and run Sections 11.1–11.8. By default it reads the `LLM-J Dataset` tab directly from the Week 4 XLSX workbook in the same folder; you can also point it to a CSV export. The notebook compares Qwen-only against a Nebius Jev-style decision model using GLM-5.3-Flash log probabilities (accept at derived two-choice confidence ≥70%; otherwise use Qwen). This is a Jev-style approximation, not a call to TypeSafe Jev. It writes CSVs for both result sets and the comparison summary.

The notebook prompts securely for a Nebius key and (when tracing is enabled) a LangSmith key; both Nebius models use the same key. It does not store credentials in the notebook or write results back to the Google Sheet. Optional provider token prices can be entered in Section 11.3 to estimate cost; without prices, cost is left blank.

Do not commit API keys, `.env` files, or other secrets.

## References and Evaluation Artifacts

### Generated artifacts

The [`outputs/` folder](outputs/) contains screenshots documenting the LangSmith traces from the evaluation runs:

- [LangSmith trace screenshot 1](outputs/LangSmith_1.png)
- [LangSmith trace screenshot 2](outputs/LangSmith_2.png)
- [LangSmith trace screenshot 3](outputs/LangSmith_3.png)

The project report, judge result exports, and AI evaluation workbook are available at the project root:

- [Customer Support Agent Evaluation project report](Customer_Support_Agent_Evaluation_Project_Report.docx)
- [LLM-only judge results](llm_j_llm_only_results.csv)
- [Nebius Jev-style hybrid judge results](llm_j_jev_hybrid_results.csv)
- [LLM-only vs. Jev-style comparison summary](llm_j_approach_comparison.csv)
- [Week 4 AI evaluation workbook](Week%204_%20AI%20Evals%20%28E-Commerce%20Customer%20Support%20Agent%29.xlsx)
