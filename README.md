# Guardrail-TCA: Temporal Consistency Auditing for Memory Poisoning Defense

A defense mechanism against memory-poisoning attacks (in the style of [MINJA](https://github.com/dsh3n77/MINJA)) on LLM agents that use persistent memory banks.

## Core Idea

Query the LLM twice for every candidate memory record:

1. Once with the **current (potentially poisoned) memory**
2. Once with an **empty memory** (clean baseline)

A reversal attack produces a *directional flip* that follows a fixed reversal map (e.g. A↔D, B↔C). Benign noise produces random, non-directional variance. TCA replaces the original Agent 1 + Agent 2 defenses at store time, and can optionally run alongside a third "Agent 3" check for additional coverage.

### Why this works when simpler defenses don't

- **Minimal-style injections** leave no linguistic fingerprint in the model's thought chain, so a defense relying on inspecting reasoning text alone is blind to them.
- If the injected thought is internally consistent with the flipped answer, a consistency-checking defense is also blind to it.
- But the **answer itself** changes directionally between the poisoned and clean calls — and that is the signal TCA is built to catch.

## Metrics Logged

| Metric | Description |
|---|---|
| `tca_flip_rate` | % of malicious records caught by TCA alone |
| `tca_fpr` | % of benign records incorrectly flagged by TCA |
| `tca_directional_hits` | Flips that match the reversal map exactly (strongest signal) |
| `tca_extra_api_calls` | Cost overhead (one extra LLM call per candidate record) |

## Notebook Structure

The notebook is organized into 11 sequential cells, meant to be run in order:

1. **Install & Clone** — clones the MINJA repo and installs dependencies
2. **Imports**
3. **Configuration** — TCA-specific knobs (enable/disable, blocking strictness, etc.)
4. **Initialize Models**
5. **Core Utilities**
6. **TCA Check + Agents 1+2 + Agent 3** — the core defense logic
7. **Data Utilities**
8. **Memory Bank & Prompt Builder**
9. **Main Runner (TCA Guardrail)**
10. **Quick Test (Single Pair)**
11. **Full Run (All 9 Pairs) + Results Table**

## Requirements

- Python 3.9+
- An OpenAI API key (set as an environment variable, e.g. `OPENAI_API_KEY`)
- See [`requirements.txt`](requirements.txt) for package dependencies

## Usage

```bash
git clone https://github.com/<your-username>/guardrail-tca.git
cd guardrail-tca
pip install -r requirements.txt
jupyter notebook guardrail-tca.ipynb
```

Run the cells in order (1 → 11). Cell 10 runs a quick single-pair sanity check; Cell 11 runs the full evaluation across all 9 pairs and prints a results table.

## Configuration

Key toggles live in the Configuration cell:

- `TCA_ENABLED` — set `False` to disable TCA and fall back to Agents 1+2 only (used as an ablation baseline)
- `TCA_BLOCK_ON_DIRECTIONAL` — if `True`, block only on exact reversal-map flips (lower false-positive rate); if `False`, block on any flip (higher recall, higher FPR)

## Related Work

This defense is designed to counter attacks described in [MINJA](https://github.com/dsh3n77/MINJA) (Memory INJection Attack), which demonstrates how an adversary can poison an LLM agent's persistent memory to manipulate future outputs.

## License

Add a license of your choice (e.g. MIT) before publishing — see [`LICENSE`](LICENSE).
