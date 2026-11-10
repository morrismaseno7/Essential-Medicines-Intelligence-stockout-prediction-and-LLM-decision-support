# Essential Medicines Intelligence: stockout prediction and LLM decision support

Two connected projects on medicine stockouts in retail pharmacy supply chains in Kenya.

1. **Stockout prediction model.** Predicts which medicines are likely to stock out at which stores.
2. **LLM decision-support assistant.** Tests how prompt design affects the safety of AI-generated stock recommendations when data is incomplete. A retrieval layer over Kenyan medicines documents is planned (see Status).

Author: Morris Maseno Sikweya, Nairobi.

## Why this matters

A stockout of an essential medicine such as an antimalarial or an antibiotic directly affects patients. Stock data in practice is often incomplete: consumption is unrecorded, lead times are missing, and zero dispensing can mean either no demand or no stock. A tool that hides those gaps is worse than no tool.

## Project 1: Stockout prediction

- **Data:** retail pharmacy dispensing, store, drug and outbreak-context CSVs. [Add source, date range, number of stores and drugs, and whether the data is shareable.]
- **Approach:** [Add features, target definition, model type, train/test split.]
- **Results:** [Add metrics with the validation method. Report the baseline you compared against.]
- **Limitations:** [Add what the model does not handle, for example demand censored while stock is out.]
- **Notebook:** `notebooks/stockout_model.ipynb`

## Project 2: Prompt experiment (E1)

**Question.** How much does prompt design change the safety of an LLM's stock recommendations when the input data has gaps?

**Setup.** Three prompts were run on gemini-3.5-flash-lite against a 5-medicine stock table.
- Prompt A: basic request for analysis and actions.
- Prompt B: "identify medicines at risk of stockout".
- Prompt C: structured output with constraints (do not invent data, flag missing data, separate calculated values from assumptions, no autonomous action, human verification).

Two scenarios: "trap" (missing consumption and lead-time values, plus an item with zero stock and zero consumption; 3 runs per prompt) and "clean" (complete data; 1 run per prompt). Each output was scored 0 to 2 for accuracy and for constraint adherence. The rubric is in `experiments/E1_scoring_rubric.md`.

**Results.**

| Scenario | Prompt | Accuracy (mean) | Constraints (mean) |
|---|---|---|---|
| Trap | A | 1.33 | 0.33 |
| Trap | B | 0.00 | 0.33 |
| Trap | C | 2.00 | 2.00 |
| Clean | A | 1.00 | 0.00 |
| Clean | B | 2.00 | 1.00 |
| Clean | C | 2.00 | 2.00 |

**Findings.**
1. Prompt B rated amoxicillin (3 days of stock, 14-day lead time) as low risk in all 3 runs. It added the on-order quantity to current stock and ignored the gap before delivery. The prompt wording alone introduced a stockout-causing error.
2. Prompts A and B invented order quantities and acted on missing lead times. Prompt C declined to classify and asked for the missing data in every run.
3. Zero stock with zero consumption was read as "no demand" by B. C flagged it as a possible stockout. Recorded consumption is censored when stock is out, which is also the central difficulty for Project 1.
4. All prompts calculated correctly. The differences were in judgement under uncertainty.

**Weaknesses in prompt C.** Risk labels for the zero-stock item varied between runs, borderline items received different actions, and on-order quantities were missing from the table. Prompt C2 (`prompts/prompt_C2.md`) addresses these with a fixed label set and mechanical thresholds. **C2 has not been tested yet.**

**Limitations.** One model, one scenario, 3 runs per trap cell, 1 per clean cell, scored by the author. The results are indicative and not statistically tested.

## Status and next steps

- [x] Stockout prediction model
- [x] Prompt experiment E1 (scored, written up)
- [ ] Run prompt C2 on the trap scenario (3 runs) and compare label consistency
- [ ] Retrieval over Kenyan essential medicines documents, so thresholds and clinical priority come from sources and not from model assumptions
- [ ] Link the assistant to stockout model outputs, so it can explain predicted risk per store and drug

## Repository layout

```
notebooks/stockout_model.ipynb
experiments/experiment_log_scored.csv
experiments/E1_scoring_rubric.md
experiments/E1_findings.md
prompts/prompt_C2.md
README.md
```

## Reproducing

[Add Python version, required libraries, how to run the notebook, and how to run the prompt experiment, including which API or tool was used. State if the dataset cannot be shared.]
