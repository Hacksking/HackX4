# BACKEND.md — Build Spec (FastAPI)

## Folder Structure
```
backend/
  main.py              # FastAPI app, route registration, CORS
  db.py                # SQLite connection + init_db()
  models.py            # Pydantic request/response models
  routers/
    upload.py
    transactions.py
    dashboard.py
    cashflow.py
    goals.py
    liabilities.py
    alerts.py
    simulate.py
  engines/
    categorizer.py      # rules + LLM fallback
    recurring.py         # subscription detector
    cashflow_engine.py
    goals_engine.py
    liability_engine.py
    alerts_engine.py
  ai/
    llm_client.py        # wraps Claude/GPT API calls
    prompts.py            # prompt templates (see AI_INTEGRATION.md)
  data/
    seed_transactions_user1.csv
    seed_transactions_user2.csv
```

## Build Order
1. `db.py` — init SQLite, run `CREATE TABLE IF NOT EXISTS` from `DATABASE.md`
2. `routers/upload.py` — parse CSV (pandas), insert into `transactions`
3. `engines/categorizer.py`:
   - Step 1: rules dict `{ "swiggy": "Food", "zomato": "Food", "netflix": "Subscription", ... }`
   - Step 2: for unmatched merchants, batch-call LLM with structured JSON prompt
     (see `AI_INTEGRATION.md`)
4. `engines/recurring.py` — group transactions by `merchant`, flag as recurring if
   same merchant appears ≥2 times with similar amount (±10%) at ~monthly intervals
5. `engines/cashflow_engine.py`:
   - sum recurring inflows (salary) minus recurring outflows (rent, EMI, subs)
   - add average discretionary spend/day × remaining days in month
   - output daily projected balance array
6. `engines/goals_engine.py`:
   - `monthly_contribution = (target_amount - current_amount) / months_remaining`
7. `engines/liability_engine.py`:
   - `true_annual_cost = balance * (interest_rate / 100)`
   - sort descending by `interest_rate` (avalanche method) → assign `priority_rank`
8. `engines/alerts_engine.py`:
   - overspend: category spend this month > 1.3x average of last 3 months
   - large upcoming debit: any known recurring debit > 20% of avg monthly income
   - unused subscription: recurring merchant with no matching "usage" transaction
     in 30+ days (for demo, can hardcode based on category = Entertainment)
9. `routers/simulate.py` — clone user's current data in memory, apply delta
   (new EMI / rent change / income change), rerun `cashflow_engine` +
   `goals_engine`, return before/after

## Rules
- Every engine function must be pure (input data → output numbers), no side effects,
  so they're independently testable.
- LLM is called only from `categorizer.py` and the narration step in `alerts_engine.py`
  / `simulate.py` — never for arithmetic.
- Enable CORS for `http://localhost:5173` (Vite default) from hour 1.
