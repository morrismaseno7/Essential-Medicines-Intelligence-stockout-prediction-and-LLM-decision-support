# Experiment 1: Prompt design for stockout risk analysis on incomplete data

**Setup.** Three prompts (A: basic, B: "identify medicines at risk", C: structured with constraints) were run on gemini-3.5-flash-lite against a 5-medicine stock table. The "trap" version had missing consumption and lead-time values and an item with zero stock and zero consumption; the "clean" version had complete data. Trap: 3 runs per prompt. Clean: 1 run per prompt. Outputs were scored 0-2 for accuracy and for constraint adherence (rubric in `E1_scoring_rubric.md`).

| Scenario | Prompt | Accuracy (mean) | Constraints (mean) |
|---|---|---|---|
| Trap | A | 1.33 | 0.33 |
| Trap | B | 0.00 | 0.33 |
| Trap | C | 2.00 | 2.00 |
| Clean | A | 1.00 | 0.00 |
| Clean | B | 2.00 | 1.00 |
| Clean | C | 2.00 | 2.00 |

**Findings**
1. Prompt B rated amoxicillin (3 days of stock, 14-day lead time) as low risk in all 3 runs. It added the on-order quantity to current stock and ignored the gap before delivery. Wording alone introduced an unsafe error.
2. Prompts A and B filled gaps with invented figures (order quantities, example consumption rates) and acted on missing lead times. Prompt C declined to classify and asked for the missing data in all 3 runs.
3. Zero stock with zero recorded consumption was read as "no demand" by B. C flagged it as a possible stockout. Recorded consumption is censored when stock is out, which is also the central difficulty for the stockout model.
4. All prompts did the arithmetic correctly. The differences were in judgement and handling of uncertainty.

**Known weaknesses in prompt C**
- The risk label for zero-stock/zero-consumption items varied between runs.
- Borderline items (Metformin) received different actions across runs.
- On-order quantities were not shown in the table.

**Limitations.** One model, one scenario, 3 runs per trap cell, single-author scoring. Results are indicative only and not statistically tested.

**Next step.** Prompt C2 (`prompt_C2.md`) addresses the three weaknesses. RAG over Kenyan medicines guidance is planned next so that thresholds and clinical priority come from documents rather than model assumptions.
