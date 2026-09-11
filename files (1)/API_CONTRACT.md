# API_CONTRACT.md — Frontend ↔ Backend Contract

Base URL (dev): `http://localhost:8000`

---

### POST `/upload`
Upload a CSV of transactions for a user.
**Request:** multipart/form-data, field `file`, query param `user_id`
**Response:**
```json
{ "status": "success", "rows_imported": 128 }
```

---

### GET `/transactions?user_id=1`
**Response:**
```json
[
  { "id": 1, "date": "2026-08-03", "merchant": "Swiggy", "amount": -450,
    "category": "Food", "type": "discretionary", "is_recurring": 0 }
]
```

---

### GET `/dashboard/summary?user_id=1`
Aggregated spend for charts.
**Response:**
```json
{
  "total_income": 85000,
  "total_expense": 62000,
  "by_category": { "Food": 8000, "Rent": 20000, "EMI": 15000 },
  "recurring_subscriptions": [
    { "merchant": "Netflix", "amount": 649, "last_used_days_ago": 45 }
  ]
}
```

---

### GET `/cashflow?user_id=1`
**Response:**
```json
{
  "projected_month_end_balance": 14200,
  "daily_projection": [
    { "date": "2026-09-01", "balance": 40000 },
    { "date": "2026-09-02", "balance": 39500 }
  ],
  "shortfall_warning": false
}
```

---

### POST `/goals`
**Request:**
```json
{ "user_id": 1, "name": "Emergency Fund", "target_amount": 100000,
  "target_date": "2027-03-01" }
```
**Response:** same object + `id`, plus computed field:
```json
{ "id": 3, "monthly_contribution_required": 5200 }
```

### GET `/goals?user_id=1`
Returns list of goals each with `monthly_contribution_required`.

---

### GET `/liabilities?user_id=1`
**Response:**
```json
[
  { "id": 1, "name": "Credit Card", "balance": 30000, "interest_rate": 36,
    "true_annual_cost": 10800, "priority_rank": 1,
    "reason": "Highest interest rate — clearing this first saves ₹10,800/year" }
]
```

---

### GET `/alerts?user_id=1`
**Response:**
```json
[
  { "id": 1, "type": "unused_sub", "message": "Netflix unused for 45 days, ₹649/month",
    "data": { "merchant": "Netflix", "amount": 649, "days": 45 } }
]
```

---

### POST `/simulate`
**Request:**
```json
{ "user_id": 1, "scenario": "new_emi", "amount": 12000 }
```
**Response:**
```json
{
  "before": { "projected_balance": 14200, "goal_timeline_months": 8 },
  "after": { "projected_balance": 2200, "goal_timeline_months": 15 },
  "narration": "Taking this EMI pushes your Emergency Fund goal back by 7 months."
}
```

---

## Rules for both sides
- All amounts in INR, numbers not strings.
- Dates always ISO `YYYY-MM-DD`.
- Every "recommendation" field must be accompanied by the raw numbers it's based on
  — never send prose without the backing data in the same response.
