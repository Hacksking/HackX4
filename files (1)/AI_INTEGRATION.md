# AI_INTEGRATION.md — LLM Usage Rules & Prompts

## Golden Rule
**The LLM never computes financial numbers.** All math (interest cost, monthly
contribution, projected balance) is done by deterministic Python in `engines/`.
The LLM only: (1) classifies ambiguous merchant strings, (2) turns computed
numbers into a plain-language explanation.

---

## Use 1 — Transaction Categorization (fallback only)

Called only for merchants that don't match the rules dictionary in
`categorizer.py`. Batch multiple unmatched merchants into one call.

**System prompt:**
```
You are a financial transaction classifier. Given a list of merchant names,
return ONLY a JSON array, no preamble, no markdown fences. For each merchant
return: name, category (one of: Food, Transport, Rent, EMI, Subscription,
Shopping, Utilities, Insurance, Entertainment, Healthcare, Other), and type
(one of: essential, discretionary).
```

**User prompt:**
```
["Merchant A", "Merchant B", "Merchant C"]
```

**Expected response:**
```json
[
  { "name": "Merchant A", "category": "Shopping", "type": "discretionary" }
]
```

---

## Use 2 — Recommendation Narration

Called after `liability_engine.py` or `alerts_engine.py` produces numbers.
Pass the numbers in, ask for a one-to-two sentence explanation.

**System prompt:**
```
You are a financial assistant. You will be given computed numbers. Write a
short (1-2 sentence), clear, encouraging explanation using those exact numbers.
Do not invent or alter any figure. Do not add numbers not provided to you.
```

**User prompt example:**
```
Liability: Credit Card, balance 30000, interest_rate 36%, true_annual_cost 10800.
This is the highest interest liability the user has.
```

**Expected response:**
```
Your credit card is costing you ₹10,800 a year in interest at 36% — clearing
this before your other loans will save you the most money.
```

---

## Use 3 — Simulation Narration

Called after `simulate.py` reruns cashflow/goals engines with a delta applied.

**User prompt example:**
```
Before: projected_balance 14200, goal_timeline_months 8.
After (new EMI of 12000/month): projected_balance 2200, goal_timeline_months 15.
```

**Expected response:**
```
Taking on this ₹12,000 EMI drops your monthly buffer to ₹2,200 and pushes your
goal timeline back by 7 months.
```

---

## Implementation Notes
- Use structured/JSON-mode output wherever possible so responses are directly
  parseable — never regex free text.
- Wrap every LLM call in try/except; if it fails or returns malformed JSON,
  fall back to a simple template string built from the raw numbers so the demo
  never breaks on an API hiccup.
- Keep `llm_client.py` as the single wrapper function so the model/provider can
  be swapped in one place if needed.
