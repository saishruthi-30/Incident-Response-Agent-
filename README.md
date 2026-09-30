# 🧠 Incident Response Agent

An IT incident response agent with **persistent memory**. Instead of generating generic advice for every incident, it recalls genuinely similar past incidents, reasons over them, and gets smarter every time an incident is resolved — powered by [Hindsight](https://hindsight.vectorize.io/) for memory and [Groq](https://groq.com/) for fast LLM inference.

## Why this exists

Most "AI incident response" demos are just a chatbot wrapped around an LLM — every answer is generic, and nothing is remembered between incidents. This agent is different: every diagnosis is grounded in real, retrieved evidence from past incidents, and every resolved incident becomes a memory the agent can draw on next time.

```
Incident → Understand → Recall similar past incidents (Hindsight) → Reason (Groq)
    → Diagnosis + evidence → Human resolves → Post-mortem → Retain → remembered forever
```

## What it does

- **Diagnoses IT incidents** with structured output: severity, likely root causes, investigation steps, recommended action, and escalation target
- **Grounds every diagnosis in retrieved memory** — cites the specific past incident(s) it's drawing from, or honestly says "no closely related past incidents found" instead of hallucinating a match
- **Learns from resolutions** — after a human resolves an incident, the post-mortem is saved to memory, and the next similar incident recalls that lesson
- **Ships with a working demo UI** (Gradio) — type or click an example incident, watch what the agent recalls, resolve it, and see it learn live

## How it's built

| Layer | Tool |
|---|---|
| Memory (retain / recall / reflect) | [Hindsight](https://hindsight.vectorize.io/) |
| Reasoning | Groq (`openai/gpt-oss-120b`) |
| Demo bank | 30 curated, realistic incidents (auth, database, network, deployment, security, etc.) |
| Evaluation dataset | [Incident Response Log](https://www.kaggle.com/datasets/vipulshinde/incident-response-log) — ServiceNow-derived, 24,918 real incidents |
| UI | Gradio |

## Project structure

This is a single notebook, organized in stages:

1. **Stage 1 — Prove the loop works.** Dataset → memory documents → bare LLM reasoning (no memory) → retrieval-augmented reasoning → side-by-side comparison.
2. **Stage 2 — Real Hindsight integration.** Swap manual retrieval for Hindsight `retain`/`recall`, and add the resolve → post-mortem → retain learning loop.
3. **Improvements.**
   - Validated, structured JSON diagnosis output
   - A second memory bank built from the real Kaggle dataset (200 real incidents)
   - An honest evaluation: does Hindsight-backed memory actually beat a simple baseline on real data?
   - `reflect` — synthesizing patterns across multiple memories, not just retrieving them
4. **Interactive UI.** A Gradio app — live agent tab (analyze, memory panel, resolve & teach) and an evidence tab (the evaluation results).

## Running it

### In Colab
Open the notebook in Google Colab and run the cells top to bottom. You'll be prompted for a Groq API key and a Hindsight API key (`getpass`, never saved to the notebook).

### Locally (VS Code)
```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```
Copy `.env.example` to `.env` and add your keys. Download the [dataset](https://www.kaggle.com/datasets/vipulshinde/incident-response-log) and place it at `data/incident_event_log.csv`. Then open the notebook in VS Code (Python + Jupyter extensions) and run cell by cell, or run the equivalent `.py` script directly.

The site launches at `http://127.0.0.1:7860`.

## Honest limitations

- The real Kaggle dataset is anonymized (categories and symptoms are numeric codes, not real text), so semantic retrieval on it is weak. The evaluation tab shows this plainly — a simple frequency-based rule beats memory-based retrieval on this specific dataset. The curated 30-incident demo bank, which has real descriptive text, is where the memory advantage is actually visible.
- The Gradio share link and local server only run while the notebook/script is active — this isn't deployed as a permanent hosted service (yet).

## Demo script

1. **Auth/2FA failure** → Analyze → point at the Agent Memory panel and the retrieved match
2. **Mobile crash** → Analyze (weak/no match) → Resolve & teach → save
3. Analyze a similar mobile-crash incident → the agent now recalls the lesson it just learned
4. **Printer toner** (unrelated incident) → agent correctly says no related incident exists, instead of inventing one
5. **Evidence tab** → show the real-data evaluation, and be upfront about where memory helps (learning, explanation, novel-pattern recognition) versus where a lookup table is competitive

## Credits

Built for [hackathon name] using Hindsight (required tech) and Groq. Dataset: Incident Response Log (Kaggle / UCI ServiceNow event log).
